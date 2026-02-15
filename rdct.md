# NullCon 2026 CTF Write-up: PDF Forensics Challenge

**Category:** Forensics / Steganography  
**Challenge Name:** "Lazy Student" / Planned Flags  
**Difficulty:** Medium  

---

## Challenge Description

We were provided with a PDF document (`Planned-Flags-signed-2.pdf`) that appeared to be a draft for a CTF challenge created by a "Lazy Student" AI. The document contained redactions, black bars, and hidden text.

The goal was to uncover all 6 hidden flags.

---

## Tools Used

- **Kali Linux Terminal** (WSL)
- `qpdf` (to uncompress streams)
- `grep` / `strings` (text analysis)
- `pdfimages` (image extraction)
- `zbarimg` (QR code scanning)
- `sed` / `xxd` / `perl` (stream carving and decoding)

---

## Preparation: Uncompressing the PDF

PDFs often compress their text streams (using `FlateDecode`), making simple `grep` commands ineffective.  
We first created a raw, uncompressed dump of the file to make all text readable.

```bash
qpdf --qdf --object-streams=disable Planned-Flags-signed-2.pdf raw_dump.txt
```

---

## Flag 1: The "Redacted" Text

**Location:** Section 3.1  
**Method:** Plain Text Search  

We searched for the standard flag format `ENO{` in the uncompressed text. The first flag was sitting in plain text, partially obscured by a black bar in the visual layer, but still visible in the underlying PDF content stream.

```bash
grep "ENO{" raw_dump.txt
```

**Flag 1:**

```
ENO{stability_gradient_1_disrupted}
```

---

## Flag 2: The Hidden Link

**Location:** Section 3.1 (Invisible URI)  
**Method:** Metadata Extraction  

While searching for the second flag, we noticed a pattern where the flag number is embedded in the string (e.g., `_1_`). Searching for `ENO` combined with `2` revealed a hidden hyperlink annotation that wasn't immediately visible on the page.

```bash
grep "URI" raw_dump.txt
```

Example output:

```
/URI (https://ctf.nullcon.net/ENO\{input_sanitization_2_is_overrated\})
```

**Flag 2:**

```
ENO{input_sanitization_2_is_overrated}
```

---

## Flag 3: The "Lazy" Duplicate

**Location:** Section 3.2  
**Method:** Plain Text Search  

The challenge description mentioned a "Lazy Student" using AI.  
Flag 3 was found in plain text in Section 3.2. Interestingly, this flag was sometimes reused as filler text in other parts of the document, serving as a red herring.

```bash
grep -C 5 "third flag" raw_dump.txt
```

**Flag 3:**

```
ENO{semantic_3_inference_initialized}
```

---

## Flag 4: The QR Code

**Location:** Section 3.3  
**Method:** Image Extraction  

Section 3.3 contained a square "redacted" box that looked suspicious.  
We suspected it might be an image overlay.

We extracted all images from the PDF:

```bash
pdfimages -all Planned-Flags-signed-2.pdf image_extract
```

This produced:

```
image_extract-001.png
```

The image was a QR code. Scanning it with `zbarimg` revealed the fourth flag.

**Flag 4:**

```
ENO{We_should_have_an_Ontology_to_4_categorize_our_ontologies}
```

---

## Flag 5: The Digital Signature

**Location:** Page 4 (Digital Signature Metadata)  
**Method:** Object Inspection & Hex Decoding  

This was the hardest flag.

The text on Page 4 (Sections 3.8/3.9) was physically removed from the content stream, not just visually covered. However, the page contained two Digital Signature widgets (`EnoLead` and `EnoChair`).

We located the signature objects (`21 0 obj` and `24 0 obj`) in the raw dump.  
Inspecting Object `24` revealed a massive hexadecimal block inside the `/Contents` field.

```bash
sed -n '/24 0 obj/,/endobj/p' raw_dump.txt
```

The content appeared as a hex string:

```
<308204eb...>
```

We extracted and decoded it:

```bash
grep -a -A 10 "24 0 obj" raw_dump.txt \
| grep "/Contents" \
| perl -pe 's/.*<//; s/>.*//' \
| xxd -r -p \
| strings \
| grep "ENO"
```

**Flag 5:**

```
ENO{SIGN_HERE_TO_GET_ALL_FLAGS_5}
```

---

## Flag 6: PDF Metadata

**Location:** Document Properties  
**Method:** Metadata Analysis  

The final flag was hidden in the PDF's global metadata, specifically in the `Producer` field.

A simple string search revealed it:

```bash
grep "ENO" raw_dump.txt | head -n 5
```

Example output:

```
<pdf:Producer>ENO{secureflaghidingsystem76}</pdf:Producer>
```

**Flag 6:**

```
ENO{secureflaghidingsystem76}
```

---

## Summary of Flags

| # | Flag | Hidden In |
|---|------|-----------|
| 1 | ENO{stability_gradient_1_disrupted} | Plain Text |
| 2 | ENO{input_sanitization_2_is_overrated} | URI Annotation |
| 3 | ENO{semantic_3_inference_initialized} | Plain Text |
| 4 | ENO{We_should_have_an_Ontology_to_4_categorize_our_ontologies} | QR Code (Image) |
| 5 | ENO{SIGN_HERE_TO_GET_ALL_FLAGS_5} | Digital Signature (Hex) |
| 6 | ENO{secureflaghidingsystem76} | PDF Metadata |

---

## Conclusion

This challenge demonstrated multiple PDF forensics techniques:

- Inspecting raw content streams  
- Extracting embedded images  
- Analyzing digital signature objects  
- Parsing and decoding hexadecimal blobs  
- Inspecting document metadata  

It was a well-rounded exercise in practical PDF reverse engineering and steganography.
