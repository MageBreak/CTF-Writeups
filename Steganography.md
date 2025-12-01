# 🚩 LSD#4: Coordinate-Offset LSB Steganography

**Category:** Steganography (LSB)
**Difficulty:** Medium
**Score:** 50

## Summary
The flag was hidden in the Least Significant Bit (LSB) layer of a JPEG/PNG image, but it was offset to a non-standard starting position to defeat automated tools.

## The Flaw
The hidden data only existed within a **100x100 pixel square starting at coordinates (1000, 1000)**. Standard tools like `zsteg` always start reading from (0, 0), resulting in failure.

## The Solution
We wrote a custom Python script using the `Pillow` library to:
1.  Target the specific region of interest (ROI) at (1000, 1000).
2.  **Isolate Channels:** Initially, combining all RGB channels yielded garbage (noise). We determined the data was stored only in the **Red Channel's LSBs**.
3.  **Extraction:** Iterated through the 100x100 square, extracted the LSB (`R & 1`) from each Red pixel, and converted the resulting binary stream to ASCII.

**Flag:** Hero{M4YB3_TH3_L4ST_LSB?}
