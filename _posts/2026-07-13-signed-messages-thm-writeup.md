---
title: "Signed Messages — LoveNote THM Writeup"
date: 2026-07-13
layout: post
categories: [ctf, tryhackme]
tags: [rsa, signature-forgery, deterministic-crypto, pss, flask]
image:
  path: /assets/img/signedmsg-banner.png
  alt: Signed Messages THM Writeup
---

## Overview

LoveNote is a Valentine's Day themed messaging app built with Flask. Each user gets
an RSA keypair on registration, used to automatically sign messages sent through
the platform. The core vulnerability: **RSA key generation is deterministic and
seeded from the username**, meaning anyone who can reverse-engineer the seed
pattern can regenerate *any* user's private key — including `admin` — without
ever touching the server.

The end goal was to forge a valid digital signature as `admin` and have it
validate on the app's own `/verify` endpoint.

## Recon

Standard nmap scan against the box:

```bash
nmap -sV -sC <target>
```

```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-13 08:00 +0530
Nmap scan report for <target>
Host is up (0.029s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 78:38:78:e2:07:14:75:a4:28:e9:b0:a2:1e:ea:96:98 (ECDSA)
|_  256 81:b4:ef:53:4e:6e:18:e6:ca:0b:65:df:58:85:51:31 (ED25519)
5000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.10.12)
|_http-title: LoveNote - Secure Valentine's Day Messaging
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.63 seconds

```

Directory brute force with `ffuf` against port 5000 turned up the app's routes:

```bash
ffuf -u http://<target>:5000/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

```
about                  [Status: 200, Size: 15206, Words: 5413, Lines: 544, Duration: 51ms]
compose                [Status: 302, Size: 218, Words: 21, Lines: 4, Duration: 48ms]
dashboard              [Status: 302, Size: 218, Words: 21, Lines: 4, Duration: 40ms]
debug                  [Status: 200, Size: 11342, Words: 4053, Lines: 408, Duration: 38ms]
login                  [Status: 200, Size: 10999, Words: 4214, Lines: 402, Duration: 51ms]
logout                 [Status: 302, Size: 208, Words: 21, Lines: 4, Duration: 48ms]
messages               [Status: 200, Size: 12929, Words: 4872, Lines: 505, Duration: 45ms]
register               [Status: 200, Size: 12509, Words: 4825, Lines: 435, Duration: 58ms]
verify                 [Status: 200, Size: 14539, Words: 5371, Lines: 533, Duration: 57ms]
```

Two endpoints immediately stood out: `/debug` (internal logs shouldn't be public)
and `/verify` (a signature verification oracle — often a sign that the
challenge wants you to *forge* something and prove it there).

## Step 1 — Understanding the login model

The login page only asks for a **username** — no password, for any account
including `admin`. The page explained the model itself:

> LoveNote uses your username to load your cryptographic keys. Your private
> key is stored securely on our server and is used to automatically sign
> messages you send.

So authentication isn't the real gate — you can already log in as `admin` and
send messages. But you don't have `admin`'s private key, so you can't *sign*
anything as admin outside of what the server does automatically. The actual
target became: **derive admin's private key independently**.

## Step 2 — Confirming deterministic key generation

Registered a test account:

```
Username: abc
Email: abc@example.com
```

The app generated and handed back the key pair where i saved them as  `abc_priv.pem` / `abc_pub.pem`. 
After some enumeration i was unable to find anything the machine time got exhausted
machine reset (new instance, new IP), then same account was recreated with the
same username/email. The newly issued keypair saved to (`abc2_priv.pem` /
`abc2_pub.pem`) was **byte-for-byte identical** to the first. That ruled out
random key generation entirely — the keys must be a deterministic function of
some input, most likely the username (email was tested separately and shown to
have no effect). 

note: it allows us to create accounts with same mail but not the usernames so 
i concluded that it uses username for the key generation

Compared the two keys' internals to be sure:

```python
# decodekey.py
from cryptography.hazmat.primitives import serialization

with open("abc_priv.pem", "rb") as f:
    key = serialization.load_pem_private_key(f.read(), password=None)

nums = key.private_numbers()
print("n =", nums.public_numbers.n)
print("e =", nums.public_numbers.e)
print("d =", nums.d)
print("p =", nums.p)
print("q =", nums.q)
```

## Step 3 — Leaking the algorithm via `/debug`

```bash
curl -i http://<target>:5000/debug
```

The response leaked full internal implementation notes:

```
[2026-02-06 14:23:15] Using deterministic key generation
[2026-02-06 14:23:15] Seed pattern: {username}_lovenote_2026_valentine

[DEBUG] Prime derivation step 1:
[DEBUG] Converting SHA256(seed) into a large integer
[DEBUG] Checking consecutive integers until a valid prime is reached
[DEBUG] Prime p selected

[DEBUG] Prime derivation step 2:
[DEBUG] Modifying seed with PKI-related constant (SHA256(seed + b"pki"))
[DEBUG] Hashing modified seed with SHA256
[DEBUG] Converting hash into a large integer
[DEBUG] Checking consecutive integers until a valid prime is reached
[DEBUG] Prime q selected

[2026-02-06 14:23:16] RSA modulus generated from p × q
```

This is effectively pseudocode for the key generation routine:

```
seed = f"{username}_lovenote_2026_valentine"
p = next_prime( int( SHA256(seed) ) )
q = next_prime( int( SHA256(seed + "pki") ) )
n = p * q
e = 65537
d = inverse(e, (p-1)*(q-1))
```

## Step 4 — Reconstructing and validating the algorithm

Wrote a reconstruction script and validated the derived primes against the
known private key for `abc`:

```python
# psudocode.py
from hashlib import sha256
from Crypto.Util.number import bytes_to_long
import gmpy2

seed = b"abc_lovenote_2026_valentine"

p = gmpy2.next_prime(bytes_to_long(sha256(seed).digest()))
q = gmpy2.next_prime(bytes_to_long(sha256(seed + b"pki").digest()))

#the p,q values from decodekey.py
expected_p = 114003002848569837155841380031172876713215949723950919907941502596438491759753
expected_q = 60062409437952808019796582392781008249386094964693578980950617436968960102243

print(p == expected_p) # True  
print(q == expected_q) # True — algorithm confirmed
```

With `p` and `q` confirmed, a full key reconstruction script was built:

```python
# reconstructpriv.py
from Crypto.PublicKey import RSA
from Crypto.Util.number import inverse

p = 114003002848569837155841380031172876713215949723950919907941502596438491759753
q = 60062409437952808019796582392781008249386094964693578980950617436968960102243

n = p * q
e = 65537
phi = (p - 1) * (q - 1)
d = inverse(e, phi)

key = RSA.construct((n, e, d, p, q))
with open("reconstructed.pem", "wb") as f:
    f.write(key.export_key())
```

Reconstructed key for `abc` matched the server-issued key exactly. Algorithm
confirmed and generalized to any username.

## Step 5 — Generating admin's private key

Simply swapped the seed to `admin`'s username:

```python
# adminprivkeygen.py
from Crypto.PublicKey import RSA
from Crypto.Util.number import inverse, bytes_to_long
from hashlib import sha256
import gmpy2

seed = b"admin_lovenote_2026_valentine"

p = int(gmpy2.next_prime(bytes_to_long(sha256(seed).digest())))
q = int(gmpy2.next_prime(bytes_to_long(sha256(seed + b"pki").digest())))

n = p * q
e = 65537

phi = (p - 1) * (q - 1)
d = int(inverse(e, phi))

key = RSA.construct((n, e, d, p, q))

print("n =", key.n)
print("e =", key.e)
print("n bit length =", key.n.bit_length())

with open("reconstructed-admin-priv.pem", "wb") as f:
    f.write(key.export_key())
print("Private key written to reconstructed-admin-priv.pem")
print(key.export_key().decode())
```

Output:

```
n = 85126331932157837525567854053245460499291064536555820275957255468949993697387108905637573364855242718285589546535798832668842941372737201002956840568107
e = 65537
n bit length = 505
Private key written to reconstructed-admin-priv.pem
-----BEGIN RSA PRIVATE KEY-----
MIIBNgIBAAJAAaAWvmsqFsYJ5w1P4MD7NKiuZxsKi2zoHUjT4uFoKxaN2FV4y+rO
MrY07Ab75kNiRoOPxSXMH2diq1jHFa0FKwIDAQABAkAAyd0e5rjVsaCOQuwU2ytE
Ye2ywfC8scpuoq2Bbd/uv3tBn9rh3ZlOpbUV/lemOezavXw/I7Aq0puhfFt5ngYx
AiAKl/W1JbCsfrrP1jLlVAK+LvpGNpX5eFAUZI0f/u+Q0wIgJ0cEB00LPOQpnPMV
G4jX+EQPDGA92OWIaSTXk9BEw0kCIAL8gaS2Yk7OTwWGIcTqYPeSMLWYb8Dq/NAy
5GHPsWNHAiAht4nSxqWGAQtj6xxMhb14JtyQMDIHdosST4ksH5ZX2QIgAfoag6un
sSO+nSMlZhnsKG7Xh3FmWLsSjwU6jcdt1fI=
-----END RSA PRIVATE KEY-----

```

Despite the site's marketing claiming "RSA-2048" everywhere, the actual modulus
was ~505 bits — consistent with what the `abc` test key showed. The "2048" was
never real; the deterministic generation used far smaller primes than a
production system would.

Now holding a fully valid admin private key, generated entirely offline,
without any server compromise — just from knowing the username and the leaked
seed format.

## Step 6 — Understanding the actual objective: `/verify`

Note: from here i took claude help since i was unable Understand what to do next so claude helped in generating admin signature

The `/about` page described the app's signing/verification model:

> 1. **Key Generation** — RSA-2048 keypair generated per account.
> 2. **Message Signing** — message is SHA-256 hashed, then the hash is
>    encrypted with the sender's private key to form a signature.
> 3. **Signature Verification** — recipients decrypt the signature with the
>    sender's public key and compare it to a fresh hash of the message.
>
> Stack: Python, Flask, `cryptography`, RSA-2048, SHA-256, **PSS padding**.

`/verify` is a form taking `username`, `message`, and a hex `signature`, and
checking whether the signature validates against that username's public key.
This is the actual win condition: **forge a signature over a message of your
choosing, as `admin`, using the reconstructed key — and get `/verify` to
confirm it's valid.**

## Step 7 — Signature scheme trial and error

First attempt used PKCS1v1.5 + SHA-256:

```python
from Crypto.Signature import pkcs1_15
from Crypto.Hash import SHA256

h = SHA256.new(b"pwned by spirit")
sig = pkcs1_15.new(key).sign(h)
```

This produced a signature, but `/verify` rejected it:

```
Signature Invalid
```

Cross-checking against `/about`, the app explicitly stated **PSS padding**,
not PKCS1v1.5. Switching schemes:

```python
from Crypto.Signature import pss
sig = pss.new(key).sign(h)
```

This raised an exception:

```
TypeError: DigestInfo is too long for this RSA key (83 bytes).
```

(That specific error actually came from a stray PKCS1v1.5 attempt with a
larger/incompatible key elsewhere in testing — the deeper issue for PSS on
this key was the *default salt length*.)

### Why salt length mattered

PSS combines the message hash with a salt before encoding. The maximum salt
length is constrained by the modulus size:

```
max_salt_len = modulus_bytes - hash_len - 2
             = 64 - 32 - 2
             = 30 bytes
```

PyCryptodome's default PSS salt length (`hash_len`, i.e. 32 bytes for SHA-256)
didn't fit inside this small, deterministically-generated 505-bit modulus.
The server-side salt length was unknown and had to be brute-forced.

```python
from Crypto.PublicKey import RSA
from Crypto.Signature import pss
from Crypto.Hash import SHA256

key = RSA.import_key(open("reconstructed-admin-priv.pem", "rb").read())
message = b"pwned by spirit"
h = SHA256.new(message)

for salt_len in [0, 8, 16, 20, 29, 30]:
    try:
        sig = pss.new(key, salt_bytes=salt_len).sign(h)
        print(f"salt_len={salt_len}: {sig.hex()}")
    except Exception as e:
        print(f"salt_len={salt_len}: FAILED - {e}")
```

Each candidate signature was submitted to `/verify`:

```bash
#!/bin/bash
sigs=(
  "00c3878928105dba..."
  "015ae3a9dcfe1386..."
  "009107fd6e083c92..."
  "008aa1248a572276..."
  "001e2736031e956f..."
)

for sig in "${sigs[@]}"; do
  curl -s http://<target>:5000/verify \
    -d "username=admin" \
    --data-urlencode "message=pwned by spirit" \
    --data-urlencode "signature=$sig" \
    | grep -o "Signature Valid\|Signature Invalid"
done
```

The signature generated with `salt_bytes=29` (the maximum allowable salt for
this modulus) came back:

```
Signature Valid
```

### OUTPUT 
```bash
(venv) [spirit@archlinux lafb2026e8]$ cat brute.py  
#!/bin/bash 
sigs=( 
  "00c3878928105dbab6e4a2a26fbfe6446eb9c245d63574cb32e465ef70f9a71364e3bc9fbbb5be756e19c74749d73bfdabb96aaf582ad037cfe173335dbddb38" 
  "015ae3a9dcfe1386467711acd901b4dec9378080ef1fa977e761843871b0651edc04f86bda95b3b60361cc8fa2e14cdfd08e14e1e9616bc753c4f34ad868210b" 
  "009107fd6e083c92782284acfc94bbdff1f5c25860131a071d18947642926f40c7cfb24e0da9565097a81e0dbf162071f5c64ec5fe13bd2daf7e8a37c5fab760" 
  "008aa1248a57227688f54994cafad071757de6251b74560b6d547159910d602fc8c7779e48ebda7d87b4f7841ceb7ec5cbb663414d96c69dbdb5cdb120c958f7" 
  "001e2736031e956fe588daf589d65da49c90b98b3765e7c5939e41b3f4e24a0fc2882babdddc6ba03b6c889392b2900f53318e96c4927191dd64e5112efa7c8a" 
) 
 
for sig in "${sigs[@]}"; do 
  echo "Testing salt sig: ${sig:0:16}..." 
  curl -s http://10.49.128.53:5000/verify \ 
    -d "username=admin" \ 
    --data-urlencode "message=pwned by spirit" \ 
    --data-urlencode "signature=$sig" | grep -o "Signature Valid\|Signature Invalid" 
  echo "---" 
done 
(venv) [spirit@archlinux lafb2026e8]$ bash brute.py  
Testing salt sig: 00c3878928105dba... 
Signature Invalid 
--- 
Testing salt sig: 015ae3a9dcfe1386... 
Signature Invalid 
--- 
Testing salt sig: 009107fd6e083c92... 
Signature Invalid 
--- 
Testing salt sig: 008aa1248a572276... 
Signature Invalid 
--- 
Testing salt sig: 001e2736031e956f... 
Signature Valid 
--- 
```


Confirmed — a forged, cryptographically valid admin signature, submitted to
the app's own verification endpoint, produced entirely offline from just the
username.

### Flag
then immediately tried on the webpage
![Flag out put](/assets/img/signedmsg-2.png)

the flag of alltime
```
THM{PR3D1CT4BL3_S33D5_BR34K_H34RT5}
```

## Root Cause

The vulnerability had nothing to do with the signature padding scheme itself —
PSS is a sound, provably secure construction when used correctly. The actual
flaw was upstream: **RSA key material was derived deterministically from a
public, guessable value (the username) instead of from a secure random source**.

```
seed = f"{username}_lovenote_2026_valentine"
p = next_prime(SHA256(seed))
q = next_prime(SHA256(seed + "pki"))
```

Once this seed pattern leaked via `/debug`, every account's private key became
computable offline by anyone. No padding scheme, and no signature algorithm,
can protect a system where the private key itself is not actually private.

## Concepts to Remember

- **RSA signing is RSA decryption in reverse.** `sig = hash(m)^d mod n`;  
  verification is `hash(m) == sig^e mod n`. Trust flows entirely from the
  secrecy of `d`.
- **Raw RSA (no padding) is insecure** — malleability and structural attacks
  exist. That's why padding schemes (PKCS1v1.5, PSS) exist at all.
- **PKCS1v1.5** — deterministic, fixed ASN.1 DigestInfo header + padding.
  Needs enough modulus room for the full digest header; fails outright if the
  key is too small for the chosen hash.
- **PSS (Probabilistic Signature Scheme)** — hash + random salt + MGF1 mask.
  More modern, provably secure, but has a hard **salt length constraint**:
  `saltLen ≤ k - hLen - 2` where `k` = modulus size in bytes, `hLen` = digest
  size. Undersized/non-standard moduli (like this challenge's ~505-bit key)
  force non-default, smaller salt lengths — worth brute-forcing when default
  library behavior throws size errors.
- **Deterministic key generation is a critical anti-pattern.** Any input to
  key generation that an attacker can predict or supply (usernames, emails,
  timestamps, weak seeds) destroys the entire security model, regardless of
  how strong the downstream cryptographic primitives are.
- **Debug/verbose logging endpoints are a serious information-disclosure
  risk** — `/debug` here directly leaked the exact algorithm pseudocode
  needed to break the system.

## Tools Used

- `nmap` — service enumeration
- `ffuf` + SecLists — directory brute force
- `curl` — manual endpoint interaction, `/debug`, `/verify` testing
- `Caido` — HTTP history / request replay
- Python: `pycryptodome` (`Crypto.PublicKey.RSA`, `Crypto.Signature.pss` /
  `pkcs1_15`, `Crypto.Hash.SHA256`), `cryptography` (key introspection),
  `gmpy2` (`next_prime`)
