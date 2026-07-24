# 🚩 CTF Write-ups

Challenge write-ups, methodology, and solution scripts from CTF competitions I've
played with **[Team 1nf1n1ty](https://ctftime.org/)** (Top 5 in India on CTFtime).

My focus is **detection engineering and DFIR**, and much of what I enjoy in CTFs feeds
directly into that: forensics, artifact analysis, signal and file-format work, and
reverse engineering. The offensive challenges here are just as useful from the blue
side — understanding exactly how an attack works is what lets me write detections for it.

Each write-up includes the challenge description, the reasoning, the payloads or scripts
used, and the key takeaway.

---

## 🔍 Forensics, Steganography & Signal Analysis

The work closest to my DFIR focus — pulling evidence out of files, captures, and signals.

- [rdct — PDF Forensics](rdct.md) · stream decompression, embedded-image carving, digital-signature and metadata extraction
- [UART — Logic-Analyzer Signal Decoding](UART.md) · decoding a Sigrok capture to recover a UART transmission
- [Steganography — Coordinate-Offset LSB](Steganography.md) · least-significant-bit image steganography
- [emoji — Unicode Steganography](emoji.md) · data hidden in Unicode variation selectors

## 🔬 Reverse Engineering & Program Logic

- [The Chef's Secret Recipe](The%20Chef's%20Secret%20Recipe.md) · dynamic analysis
- [Seen — Invisible Logic](Seen.md) · reversing a custom PRNG verifier hidden in invisible Unicode

## 🌐 Web & Application Security

- [Shell — RCE via CVE-2021-22204](Shell.md) · ExifTool remote code execution
- [4lld4y — Prototype Pollution RCE](4lld4y.md) · Happy-DOM prototype pollution to RCE
- [Tomwhat — Cross-Context Session Pollution](Tomwhat.md)
- [revoked — SQLi & Logic Flaw](revoked.md)
- [Days Of Future Past](Apoorv-Days-Of-Future-Past.md) · web / crypto

## 🔐 Cryptography

- [Pasty — Forging a Homebrew Signature](Pasty.md) · key-reuse flaw in a custom MAC; chosen-plaintext key recovery and forgery

## 🌍 Network

- [DiNoS — DNS](DiNoS.md) · recovering a flag hidden across DNS TXT records

## ⚙️ System & Privilege Escalation

- [Movie Night — SSH Privilege Escalation](Movie%20Night.md)
- [Neverland — Git Hook Privilege Escalation](Neverland.md)
- [PVE — Algorithmic Bot](PVE.md)

---

*More write-ups added as I play. Repos for my detection-engineering and DFIR work
(detection lab, Sigma rules, ML-based detection) are pinned on my
[profile](https://github.com/MageBreak).*
