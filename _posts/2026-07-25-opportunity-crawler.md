---
title: "[Opportunity Crawler] - I Built a Bot That Scrapes Cybersecurity Jobs and Sends Them to My Telegram Every Day"
date: 2026-07-25
layout: post
categories: [projects]
tags: [python, automation, cybersecurity, telegram, github-actions, groq, ai, job-hunting, open-source]
image:
  path: /assets/img/opportunity_crawler_banner.png
hide_banner: true
---
---
<svg width="100%" viewBox="0 0 680 340" role="img" xmlns="http://www.w3.org/2000/svg">
<title>Opportunity Crawler banner</title>
<desc>Animated banner showing a terminal-style pipeline from scraper to Telegram, with scrolling job titles and AI scores</desc>

<defs>
  <clipPath id="feedclip">
    <rect x="30" y="200" width="290" height="112"/>
  </clipPath>
  <style>
    .oc-bg { fill: #0d1117; }
    .oc-grid { stroke: #1e2a1e; stroke-width: 0.5; fill: none; }
    .oc-panel { fill: #161b22; stroke: #238636; stroke-width: 1; }
    .oc-panel-blue { fill: #161b22; stroke: #1f6feb; stroke-width: 1; }
    .oc-panel-purple { fill: #161b22; stroke: #8b5cf6; stroke-width: 1; }
    .oc-lg { fill: #3fb950; font-family: monospace; font-size: 11px; }
    .oc-lb { fill: #58a6ff; font-family: monospace; font-size: 11px; }
    .oc-lp { fill: #a78bfa; font-family: monospace; font-size: 11px; }
    .oc-lx { fill: #8b949e; font-family: monospace; font-size: 10px; }
    .oc-lw { fill: #e6edf3; font-family: monospace; font-size: 11px; }
    .oc-title { fill: #e6edf3; font-family: monospace; font-size: 22px; font-weight: bold; }
    .oc-sub { fill: #58a6ff; font-family: monospace; font-size: 12px; }
    .oc-s9 { fill: #3fb950; font-family: monospace; font-size: 10px; font-weight: bold; }
    .oc-s2 { fill: #f85149; font-family: monospace; font-size: 10px; font-weight: bold; }
    .oc-cursor { fill: #3fb950; }
    @keyframes oc-blink { 0%,100%{opacity:1} 50%{opacity:0} }
    @keyframes oc-scroll { 0%{transform:translateY(0)} 100%{transform:translateY(-120px)} }
    @keyframes oc-dash { to{stroke-dashoffset:-20} }
    @keyframes oc-ping { 0%{r:4;opacity:1} 100%{r:12;opacity:0} }
    .oc-blink { animation: oc-blink 1s step-end infinite; }
    .oc-scroll { animation: oc-scroll 6s linear infinite; }
    .oc-flow { stroke: #238636; stroke-width: 1.5; fill: none; stroke-dasharray: 4 2; animation: oc-dash 1.2s linear infinite; }
    .oc-ping { animation: oc-ping 1.5s ease-out infinite; fill: none; stroke: #58a6ff; stroke-width: 1.5; }
  </style>
</defs>

<rect width="680" height="340" class="oc-bg"/>

<line x1="0" y1="40" x2="680" y2="40" class="oc-grid"/>
<line x1="0" y1="80" x2="680" y2="80" class="oc-grid"/>
<line x1="0" y1="120" x2="680" y2="120" class="oc-grid"/>
<line x1="0" y1="160" x2="680" y2="160" class="oc-grid"/>
<line x1="0" y1="200" x2="680" y2="200" class="oc-grid"/>
<line x1="0" y1="240" x2="680" y2="240" class="oc-grid"/>
<line x1="0" y1="280" x2="680" y2="280" class="oc-grid"/>
<line x1="80" y1="0" x2="80" y2="340" class="oc-grid"/>
<line x1="200" y1="0" x2="200" y2="340" class="oc-grid"/>
<line x1="360" y1="0" x2="360" y2="340" class="oc-grid"/>
<line x1="520" y1="0" x2="520" y2="340" class="oc-grid"/>

<rect x="0" y="0" width="680" height="36" fill="#161b22"/>
<circle cx="20" cy="18" r="6" fill="#f85149"/>
<circle cx="40" cy="18" r="6" fill="#d29922"/>
<circle cx="60" cy="18" r="6" fill="#3fb950"/>
<text x="340" y="23" text-anchor="middle" class="oc-lx">opportunity-crawler — main.py</text>

<text x="40" y="80" class="oc-title">🔍 opportunity-crawler</text>
<text x="40" y="100" class="oc-sub">daily cybersecurity alerts → telegram  |  free  |  github actions</text>

<rect x="30" y="118" width="100" height="46" rx="4" class="oc-panel"/>
<text x="80" y="136" text-anchor="middle" class="oc-lg">scraper.py</text>
<text x="80" y="150" text-anchor="middle" class="oc-lx">11 sources</text>
<text x="80" y="161" text-anchor="middle" class="oc-lx">~260 raw</text>

<line x1="130" y1="141" x2="165" y2="141" class="oc-flow"/>
<polygon points="162,137 170,141 162,145" fill="#238636"/>

<rect x="170" y="118" width="100" height="46" rx="4" class="oc-panel"/>
<text x="220" y="136" text-anchor="middle" class="oc-lg">filter.py</text>
<text x="220" y="150" text-anchor="middle" class="oc-lx">hard-exclude</text>
<text x="220" y="161" text-anchor="middle" class="oc-lx">+ keyword</text>

<line x1="270" y1="141" x2="305" y2="141" class="oc-flow"/>
<polygon points="302,137 310,141 302,145" fill="#238636"/>

<rect x="310" y="118" width="100" height="46" rx="4" class="oc-panel-purple"/>
<text x="360" y="136" text-anchor="middle" class="oc-lp">groq ai</text>
<text x="360" y="150" text-anchor="middle" class="oc-lx">score 1–10</text>
<text x="360" y="161" text-anchor="middle" class="oc-lx">threshold ≥7</text>

<line x1="410" y1="141" x2="445" y2="141" class="oc-flow"/>
<polygon points="442,137 450,141 442,145" fill="#238636"/>

<rect x="450" y="118" width="96" height="46" rx="4" class="oc-panel-blue"/>
<text x="498" y="136" text-anchor="middle" class="oc-lb">dedup.py</text>
<text x="498" y="150" text-anchor="middle" class="oc-lx">10-day</text>
<text x="498" y="161" text-anchor="middle" class="oc-lx">memory</text>

<line x1="546" y1="141" x2="578" y2="141" class="oc-flow"/>
<polygon points="575,137 583,141 575,145" fill="#238636"/>

<rect x="583" y="118" width="70" height="46" rx="4" class="oc-panel-blue"/>
<text x="618" y="136" text-anchor="middle" class="oc-lb">telegram</text>
<circle cx="618" cy="152" r="4" fill="#58a6ff"/>
<circle cx="618" cy="152" r="4" class="oc-ping"/>

<rect x="30" y="182" width="290" height="130" rx="4" fill="#0d1117" stroke="#30363d" stroke-width="1"/>
<rect x="30" y="182" width="290" height="18" rx="4" fill="#161b22"/>
<text x="40" y="195" class="oc-lx">$ live feed — today's matches</text>

<g clip-path="url(#feedclip)">
  <g class="oc-scroll">
    <text x="40" y="218" class="oc-lg">✓</text>
    <text x="55" y="218" class="oc-lw">SOC Analyst L1 — Airtel Digital</text>
    <text x="312" y="218" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="232" class="oc-lg">✓</text>
    <text x="55" y="232" class="oc-lw">Cyber Security Intern — Rubrik</text>
    <text x="312" y="232" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="246" class="oc-lg">✓</text>
    <text x="55" y="246" class="oc-lw">VAPT Appsec Red Teaming — KPMG</text>
    <text x="312" y="246" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="260" class="oc-lg">✓</text>
    <text x="55" y="260" class="oc-lw">Incident Response Associate — UnitedLex</text>
    <text x="312" y="260" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="274" class="oc-lx">✗</text>
    <text x="55" y="274" class="oc-lx">Senior Security Manager — Accenture</text>
    <text x="312" y="274" text-anchor="end" class="oc-s2">[2]</text>

    <text x="40" y="288" class="oc-lx">✗</text>
    <text x="55" y="288" class="oc-lx">Security Analyst — Alignerr (annotation)</text>
    <text x="312" y="288" text-anchor="end" class="oc-s2">[2]</text>

    <text x="40" y="302" class="oc-lg">✓</text>
    <text x="55" y="302" class="oc-lw">SOC Analyst L1 — TCS</text>
    <text x="312" y="302" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="316" class="oc-lg">✓</text>
    <text x="55" y="316" class="oc-lw">Graduate Trainee SOC — QAD</text>
    <text x="312" y="316" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="330" class="oc-lx">✗</text>
    <text x="55" y="330" class="oc-lx">IT Security Analyst II — Stefanini</text>
    <text x="312" y="330" text-anchor="end" class="oc-s2">[2]</text>

    <text x="40" y="344" class="oc-lg">✓</text>
    <text x="55" y="344" class="oc-lw">Cybersecurity Intern — Medinex</text>
    <text x="312" y="344" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="358" class="oc-lg">✓</text>
    <text x="55" y="358" class="oc-lw">SOC Analyst L1 — Airtel Digital</text>
    <text x="312" y="358" text-anchor="end" class="oc-s9">[9]</text>

    <text x="40" y="372" class="oc-lg">✓</text>
    <text x="55" y="372" class="oc-lw">Cyber Security Intern — Rubrik</text>
    <text x="312" y="372" text-anchor="end" class="oc-s9">[9]</text>
  </g>
</g>

<rect x="340" y="182" width="140" height="130" rx="4" fill="#0d1117" stroke="#30363d" stroke-width="1"/>
<rect x="340" y="182" width="140" height="18" rx="4" fill="#161b22"/>
<text x="350" y="195" class="oc-lx">today's run</text>
<text x="350" y="218" class="oc-lx">raw scraped</text>
<text x="472" y="218" text-anchor="end" class="oc-lw">260</text>
<text x="350" y="234" class="oc-lx">hard-excluded</text>
<text x="472" y="234" text-anchor="end" class="oc-s2">−94</text>
<text x="350" y="250" class="oc-lx">keyword-miss</text>
<text x="472" y="250" text-anchor="end" class="oc-s2">−28</text>
<text x="350" y="266" class="oc-lx">ai-filtered</text>
<text x="472" y="266" text-anchor="end" class="oc-s2">−30</text>
<line x1="350" y1="274" x2="472" y2="274" stroke="#30363d" stroke-width="0.5"/>
<text x="350" y="290" class="oc-lg">sent to telegram</text>
<text x="472" y="290" text-anchor="end" class="oc-s9">108</text>
<text x="350" y="306" class="oc-lx">api cost</text>
<text x="472" y="306" text-anchor="end" class="oc-lw">₹0</text>
<rect x="350" y="314" width="122" height="16" rx="3" fill="#1a3320"/>
<text x="411" y="325" text-anchor="middle" class="oc-lg">⏰ 9 PM IST daily</text>

<rect x="498" y="182" width="152" height="130" rx="4" fill="#0d1117" stroke="#30363d" stroke-width="1"/>
<rect x="498" y="182" width="152" height="18" rx="4" fill="#161b22"/>
<text x="508" y="195" class="oc-lx">model chain</text>
<rect x="508" y="204" width="132" height="20" rx="3" fill="#1a1040"/>
<text x="574" y="218" text-anchor="middle" class="oc-lp">qwen/qwen3.6-27b</text>
<text x="574" y="236" text-anchor="middle" class="oc-lx">↓ rate limit? rotate</text>
<rect x="508" y="240" width="132" height="20" rx="3" fill="#0d1f2d"/>
<text x="574" y="254" text-anchor="middle" class="oc-lb">openai/gpt-oss-20b</text>
<text x="574" y="272" text-anchor="middle" class="oc-lx">↓ ssl drop? rotate</text>
<rect x="508" y="276" width="132" height="20" rx="3" fill="#0d1f2d"/>
<text x="574" y="290" text-anchor="middle" class="oc-lb">openai/gpt-oss-120b</text>
<text x="508" y="314" class="oc-lx">1K RPD × 3 models</text>
<text x="508" y="328" class="oc-lx">= 3K daily calls free</text>

<text x="40" y="328" class="oc-lg">$</text>
<rect x="51" y="315" width="8" height="14" class="oc-cursor oc-blink"/>

</svg>


---


Hey everyone,

So this started from a genuinely frustrating moment. I found out about APCSIP — a government cybersecurity internship program — after the application deadline had already passed. Not by a few days. By weeks. I just hadn't been checking the right places at the right time.

That got me thinking: how many other opportunities am I missing just because I'm not actively monitoring 10 different portals every day? LinkedIn, NCS, CDAC, NCIIPC, CERT-In, Reddit netsec hiring threads, HackerEarth — nobody has time to check all of these manually every morning.

So I built a bot to do it for me.

This post covers what it does, how it works technically, and how you can set up your own version in about 30 minutes.

---

## What It Does

Every day at 9 PM, a GitHub Actions cron job runs and:

1. Scrapes 11 sources for cybersecurity jobs, internships, CTF competitions, and government programs
2. Hard-filters obvious noise — senior roles, non-India locations, physical security jobs, AI annotation jobs disguised as "Security Analyst" roles
3. Keyword scores everything against 67 cybersecurity-specific terms
4. Sends surviving items to Groq's free LLM API, which scores each one 1–10 against my resume profile
5. Deduplicates against the last 10 days of sent items
6. Fires off Telegram messages sorted by score — best matches first, Hyderabad roles prioritized within same score

The result: every evening I get a clean batch of actually relevant opportunities I haven't seen before. No noise, no repeats, no manual work.

---

## The Pipeline

```
GitHub Actions (cron 9PM IST)
  → scraper.py      scrapes all 11 sources (~260 raw items/day)
  → filter.py       3-stage pipeline:
      Stage 1: hard_exclude()    instant, no API (~100 items dropped)
      Stage 2: keyword_score()   67 keywords, min 1 to pass (~28 miss)
      Stage 3: ai_score_groq()   Groq LLM, 1-10, threshold ≥7 (~30 filtered)
  → dedup.py        seen_urls.json, 10-day retention
  → notifier.py     Telegram Bot API, sorted by AI score
```

Typical run: 260 raw → 94 hard-excluded → 28 keyword-miss → 30 AI-filtered → ~108 sent to Telegram.

---

## Sources

| Source | What I scrape |
|--------|--------------|
| LinkedIn | SOC L1, VAPT, security analyst, intern roles — India only |
| NCS (National Career Service) | Government job portal |
| Google News via DuckDuckGo | Cybersecurity internship announcements |
| CERT-In / DSCI | Government cyber programs and challenges |
| CDAC / MeitY / NCIIPC / AICTE / NICSI | Govt portals for training and intern programs |
| HackerEarth | CTF competitions and hackathons |
| GitHub API | Security research programs |
| Reddit netsec / cybersec | Hiring megathreads |
| RSS feeds | InfoSec Jobs, HackerNews |

**Permanently disabled:** Indeed (403 forever), Internshala (fake listings), Unstop (too noisy), TimeJobs (connection always fails).

---

## Stage 1 — Hard Exclude (No API, Instant)

This runs before anything hits an API. It's pure pattern matching and kills about 40% of raw items immediately.

Things that get hard-excluded:

- **Senior/mid-level titles** — "Senior Security Analyst", "Security Manager", "SOC Analyst L2/L3", "IT Security Analyst II"
- **Non-India locations** — checks the `location` field, job title, and URL slug (because LinkedIn sometimes leaves the location field empty but encodes it in the URL like `/soc-analyst-l1-al-khobar-saudi-national-at-...`)
- **Blocked companies** — Alignerr (AI annotation jobs dressed up as "Security Operations Analyst"), ScoutIT (posts 8 identical listings with different IDs), Pinkerton (physical security firm)
- **Wrong field** — fleet operations analyst, fraud analyst, geopolitical risk analyst, marketing intern
- **Bad URLs** — Reddit hiring threads, LinkedIn posts/articles, Medium blogs, Facebook

```python
# Example: catching non-India LinkedIn roles even when location field is empty
url_lower = opp.get("url", "").lower()
if any(m.replace(" ", "-") in url_lower for m in _NON_INDIA_MARKERS):
    return True, f"non-India location in URL"
```

This caught a "SOC Analyst L1 Al Khobar Saudi National" role that kept slipping through because the scraper was setting `location='India'` (generic fallback) while the actual location was encoded in the URL slug.

---

## Stage 2 — Keyword Scoring

67 keywords covering everything from `"soc analyst"` to `"threat hunting"` to `"apcsip"`. Each keyword has a weight. Items with total score below 1 are dropped. Intentionally low threshold — let the AI decide, not keyword matching.

The keywords are split into single words (`"splunk"`, `"vapt"`, `"forensics"`) and phrases (`"security operations center"`, `"incident response"`). Single words are critical because titles like "SOC Analyst" would get zero keyword score if you only match on `"cybersecurity intern"`.

---

## Stage 3 — AI Scoring with Groq

This is where the real filtering happens. Each surviving item gets sent to a Groq LLM with the job details and my full `resume_profile.yaml`:

```
Score this opportunity for this candidate from 1-10.
Candidate: B.Tech CSE Cybersecurity, JNTUH, 3rd year.
Skills: Splunk, Burp Suite, Nmap, Python, Wireshark, Metasploit.
Experience: Web VAPT internship (1 month). SOC home lab.
Seeking: Entry-level/intern only. India only.

Opportunity:
Title: {title}
Company: {company}
Description: {description}

Respond with JSON only: {"score": N, "reason": "..."}
```

Score ≥ 7 gets sent. Score ≤ 6 is dropped.

**Why this works better than just keywords:**

A "Security Analyst" role at a company that does AI training data annotation keeps passing keyword filters — the title is perfect. But Groq sees the description mentioning "annotate security scenarios for AI model training" and scores it 2/10. Dropped.

A "CD&E-Cybersecurity-SOC L1 Support - Associate 2 Bangalore" at PwC looks weird in a keyword filter (what even is CD&E?) but Groq understands it's a SOC L1 fresher role at a Big 4 and scores it 9/10.

**Model chain (Groq free tier, round-robin):**
```python
GROQ_MODEL_CHAIN = [
    "qwen/qwen3.6-27b",
    "openai/gpt-oss-20b",
    "openai/gpt-oss-120b",
]
```

Round-robin spreads the load across all three models. If one hits a rate limit or connection drops, it automatically falls to the next. The free tier gives 1,000 requests/day per model = 3,000 total daily calls. A full run uses ~120 calls. Plenty of headroom.

One annoying thing I debugged: reasoning models (qwen3.6, gpt-oss) output a `<think>...</think>` chain before the JSON. You have to strip it before parsing:

```python
think_end = content.find("</think>")
if think_end != -1:
    content = content[think_end + 8:].strip()
```

---

## Deduplication

Every sent URL goes into `seen_urls.json` with a timestamp and the AI score:

```json
"https://in.linkedin.com/jobs/view/soc-analyst-at-airtel-digital-4430231018": {
  "t": "2026-07-25T08:20:08",
  "s": 9
}
```

Entries older than 10 days are pruned on each run. This means a listing re-surfaces after 10 days if it's still active — which is intentional, since you may have missed it or the situation changed.

In GitHub Actions, this file is committed back to the repo after each run by the bot:

```yaml
- name: Commit seen_urls.json
  run: |
    git config user.name "github-actions[bot]"
    git config user.email "github-actions[bot]@users.noreply.github.com"
    git add seen_urls.json debug_ai_scores.json debug_excluded.json || true
    git diff --cached --quiet || git commit -m "chore: update seen_urls [skip ci]"
    git push
```

The `[skip ci]` tag in the commit message prevents this commit from triggering another workflow run.

---

## What a Telegram Message Looks Like

Each notification is a separate Telegram message with the job title, company, location, source, AI score, and a direct link. They arrive sorted — score 10 first, then 9, then 8, and within the same score, Hyderabad roles come before other cities.

So the first message I see every evening is always the highest-scored, most locally relevant opportunity from that day's run.

---

## Bugs I Had to Fix

A few interesting ones from building this:

**1. Groq model deprecations causing silent failures**

Models `llama-4-scout-17b` and `qwen3-32b` were deprecated on June 17, 2026 and started returning 404. The exception handler was catching these as generic errors and returning a fallback score of 7 — so everything was passing AI filtering. Fixed by tracking dead models per run and skipping them immediately on subsequent calls.

**2. `gpt-oss-120b` returning 400 on `response_format: json_object`**

Some Groq models just don't support the JSON response format parameter. Solution: remove the parameter entirely and parse the JSON manually from the text response.

**3. SSL connection drops burning AI budget**

`[SSL: UNEXPECTED_EOF_WHILE_READING]` errors on Groq's side were hitting the generic exception handler and returning fallback score=7 instead of trying the next model. Each failed call still counted against the 120-call budget, so 4-5 SSL drops early in a run meant 30+ items never got AI-scored. Fixed by detecting SSL errors specifically and routing them to the next model in chain.

**4. Saudi Arabia roles slipping through India filter**

The LinkedIn scraper sometimes sets `location='India'` (generic fallback) even when the actual job is in Saudi Arabia. The location filter only checked the location field, so it passed. Fixed by also scanning the URL slug — LinkedIn encodes the actual location in the URL like `/soc-analyst-l1-al-khobar-saudi-national-at-...`

---

## Setting Up Your Own

The repo is public and designed to be forked. You need three things:

**1. Groq API key** — free at [console.groq.com](https://console.groq.com). 1,000 req/day per model, no card required.

**2. Telegram bot** — create one via @BotFather, get the token and your chat ID.

**3. Edit `resume_profile.yaml`** — replace my details with yours. This is what the AI scores against. Change your skills, target roles, and experience and the scoring adapts automatically.

Then fork the repo, add three GitHub Secrets (`GROQ_API_KEY`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`), and trigger a manual run from the Actions tab.

Full setup guide in the README: [github.com/arvdch/opportunity-crawler](https://github.com/arvdch/opportunity-crawler)

---

## Cost

Everything runs free:

| Service | Usage | Cost |
|---------|-------|------|
| GitHub Actions | ~18 min/day = ~540 min/month | Free (2,000 min/month limit) |
| Groq API | ~120 calls/day = ~3,600/month | Free (3,000/day limit × 3 models) |
| Telegram Bot API | unlimited messages | Free forever |
| **Total** | | **₹0/month** |

---

## What's Next

A few things I want to add:

- **Naukri.com scraper** — high volume Indian source for fresher roles, not scraped yet
- **Content-hash dedup** — catch identical job descriptions posted under different IDs (some companies do this aggressively)
- **Telegram digest mode** — group all notifications into one message instead of 100 individual ones
- **Score shown in notification** — right now you'd have to check `debug_ai_scores.json` to see the score; should just show it in the message

The filtering is in a pretty good state now. Running it for a few weeks to see what leaks through before adding new sources.

---

Repo: [github.com/arvdch/opportunity-crawler](https://github.com/arvdch/opportunity-crawler)

If you set it up and find something broken or want to add a new source, open an issue or PR.

— Aravind
