# NullCon CTF - Unicode Steganography Challenge

**Challenge:** Hidden in Plain Sight (README.md)  
**Category:** Steganography / Crypto  
**Difficulty:** Medium  

---

## 1. Initial Analysis

The challenge provided a `README.md` file that appeared to contain only a standard emoji (`💯`) and some instructions. However, the file size was larger than the visible text suggested.

Using `xxd` to inspect the hex dump revealed a sequence of 4-byte characters immediately following the emoji:

```bash
xxd README.md
```

Example output snippet:

```
f09f 92af f3a0 84b5 f3a0 84be f3a0 84bf ...
```

```
f09f 92af  -> 💯 (Visible Emoji)
f3a0 84b5  -> Invisible Character
```

Decoding the hex `f3a0 84b5` gives the Unicode code point `U+E0135`.

This falls into the **Variation Selectors Supplement** block (`U+E0100` – `U+E01EF`), which explains why the characters were invisible in the terminal.

---

## 2. Extraction Logic

The hidden characters were encoded as variation selectors offset from standard ASCII.

Example:

```
Hidden Byte: U+E0135
Base Offset: U+E0100

Math:
0xE0135 - 0xE0100 = 0x35

ASCII:
0x35 = '5'
```

So the encoding logic is:

1. Take the Unicode code point.
2. Subtract `0xE0100`.
3. Interpret the result as an ASCII character.

I wrote a script to extract all characters in the `U+E0100`–`U+E01EF` range and subtract the base `0xE0100`.

Extracted String:

```
5>?k5= :!COE>!3?4#O!CO=17!3m
```

---

## 3. Decoding the Cipher

The extracted string looked like a scrambled flag. Comparing it to the expected flag format `ENO{`:

```
Extracted: 5     (ASCII 53)
Expected:  E     (ASCII 69)
Difference: +16
```

Testing on the next character:

```
Extracted: > (ASCII 62)
62 + 16 = 78 ('N')
```

Match confirmed.

The encoding used a simple **Caesar Cipher with shift +16**.

So the full decoding process is:

1. Extract variation selector characters.
2. Subtract `0xE0100` → get intermediate ASCII.
3. Apply Caesar shift `+16`.
4. Recover the flag.

---

## 4. Solution Script

```python
#!/usr/bin/env python3

def solve():
    try:
        with open("README.md", "r", encoding="utf-8") as f:
            content = f.read()
    except FileNotFoundError:
        return

    flag = ""
    for char in content:
        # Check for Variation Selector Supplement (U+E0100 - U+E01EF)
        if 0xE0100 <= ord(char) <= 0xE01EF:
            # 1. Extract raw char (subtract base)
            raw_val = ord(char) - 0xE0100
            # 2. Apply Shift +16 to get the Flag
            flag += chr(raw_val + 16)

    print(f"FLAG: {flag}")

if __name__ == "__main__":
    solve()
```

---

## 5. The Flag

```
ENO{EM0J1S_UN1COD3_1S_MAG1C}
```
