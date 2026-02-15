# CTF Writeup: Exif Metadata Viewer (RCE via CVE-2021-22204)

---

## Challenge Overview

**Name:** Exif Metadata Viewer  
**Category:** Web / Scripting  
**Objective:** Execute arbitrary commands on the server to read the hidden `flag.txt` file.  
**Target URL:** http://chall.0xfun.org:58791  

---

## 1. Reconnaissance

The web application allows users to upload images and inspect their EXIF metadata.

Initial testing with a normal JPEG image showed that the server extracts and displays metadata fields such as:

- Artist  
- MIME Type  
- ExifTool Version Number  

From the metadata output, we identified:

```
ExifTool Version Number : 12.16
```

This version information is critical for vulnerability research.

---

## 2. Vulnerability Identification

Initial attempts at classic command injection failed. For example:

- Embedding `$(cat /etc/passwd)` in the Artist field
- Injecting shell metacharacters

The application simply reflected these payloads as literal strings, indicating no direct command injection in metadata fields.

However, **ExifTool version 12.16** is vulnerable to:

```
CVE-2021-22204
```

### About CVE-2021-22204

This is a critical **Remote Code Execution (RCE)** vulnerability in ExifTool.

Root cause:

- Improper sanitization of DjVu metadata
- Unsafe use of Perl `eval()` on user-controlled input
- Malicious DjVu structures embedded inside a file can trigger arbitrary code execution

Important detail:

Even if the uploaded file has a `.jpg` extension, ExifTool processes embedded DjVu metadata internally, making exploitation possible through a disguised image.

---

## 3. Exploitation Strategy

Crafting a malicious DjVu payload manually is complex.

Instead, we used a publicly available Python exploit script to generate a weaponized image automatically.

---

### Environment Setup

Operating System:

- Kali Linux (via WSL)

Required dependency:

```bash
sudo apt install djvulibre-bin
```

Exploit script used:

```
exploit-CVE-2021-22204.py
```

---

## 4. Command Discovery

Before retrieving the flag, we needed to determine its location.

### Attempt 1

```bash
cat flag.txt
```

Result:

```
No such file or directory
```

### Attempt 2

```bash
ls /
```

Result:

Confirmed that `flag.txt` exists in the root directory.

### Final Payload

```bash
cat /flag.txt
```

---

## 5. Exploit Execution

We generated the malicious image using the following command:

```bash
python3 exploit-CVE-2021-22204.py -c "cat /flag.txt"
```

This produced a weaponized file:

```
image.jpg
```

The file appears to be a normal JPEG but contains a malicious DjVu annotation payload.

---

## 6. Upload & Result

After uploading `image.jpg` to the target web application:

- ExifTool processed the file
- The malicious DjVu metadata triggered the Perl `eval()` vulnerability
- The command `cat /flag.txt` was executed on the server

The contents of `/flag.txt` were reflected at the top of the metadata output page.

---

## Summary

This challenge demonstrates:

- The importance of checking software versions during reconnaissance
- The dangers of unsafe deserialization and dynamic evaluation
- How file upload features can lead to full Remote Code Execution
- Real-world exploitation of **CVE-2021-22204**

It highlights why keeping third-party tools updated is critical and why user-supplied file parsing should always be treated as untrusted input.
