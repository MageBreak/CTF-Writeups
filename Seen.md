Markdown

# CTF Writeup: Flag Checker (Invisible Logic)

## Challenge Description
The challenge provides a simple HTML page with an input field and a flag checker script. On the surface, the logic seems to compare the input against a hidden data string, but the string appears to be empty or contains only whitespace.

## Initial Analysis
Inspecting the source code reveals a script that processes a string `s` composed of "invisible" characters.

### 1. The Hidden Data
The string `s` is actually made of **Unicode Variation Selectors** (range `U+FE00` to `U+FE0F`). These are non-printing characters. The script decodes these by:
* Subtracting `0xFE00` from each character to get a 4-bit nibble.
* Combining two nibbles to form one 8-bit byte.
* Storing these bytes in an array `t`.

### 2. The Verification Algorithm
The verification logic uses a custom Pseudo-Random Number Generator (PRNG) state:
* **Initial Seed:** `0x10231048`
* **Constants:** XOR with `0xA7012948` and Add `131203`.
* **Logic:** For every character `u[i]` in the input, the state `gen` is updated.
* **Check:** The script confirms that `gen & 0xFF` (the lowest byte) matches the corresponding value in the second half of the decoded array `t`.

```javascript
gen = ((gen ^ 0xA7012948 ^ u[i]) + 131203) & 0xffffffff;
if (t[u.length + i] !== (gen % 256 + 256) % 256) return "wrong";
Solution Strategy
Since the script validates the input character-by-character and we know the target bytes in t, we can reverse the 8-bit operations to recover the flag.

Reversing the Math
To find the input character u[i], we rearrange the generator update for the lowest byte:

target = (intermediate_val + 131203) & 0xFF

intermediate_val = (target - 131203) & 0xFF

input_char = intermediate_val ^ (previous_gen & 0xFF) ^ (0xA7012948 & 0xFF)

Solver Script (Python)
Python

def solve():
    # Hidden Variation Selector string
    s = "︍︃︁️︇︁︋︂︌︅︀︀︁︄︀︆︇︃︋︅︄︂︎︈︆︌︅︅︁︆︍︎︎︉︌︃︂︄︌︇️︅️︈︂︎︍︉︋︋︁︍︂︋︇︍︋︄︆︀︀︄︎︎︇︀︋︍︍︍︈︇︌︈︅︁︍︉︆︍️︅︁︀︉︂︀︈️︄︆︆︎︍︁︇︉︎︁︋️︇︆︎️︌︀︄︀︂️︌︆︎️︅︇︈︈︇︁︎︈︌︀︊️︅︇︃︈︍︀︎︈︄︇︀︉︌︇︈︍︀"
    
    vs = 0xFE00
    # Filter and decode t
    chars = [ord(c) for c in s if 0xFE00 <= ord(c) <= 0xFE0F]
    t = [((chars[i]-vs)<<4)|(chars[i+1]-vs) for i in range(0, len(chars), 2)]
    
    u_len = len(t) // 2
    check_values = t[u_len:]
    gen = 0x10231048
    flag = ""

    for target in check_values:
        # Reverse the 8-bit addition and XOR
        term = (target - 131203) & 0xFF
        char_code = term ^ (gen & 0xFF) ^ (0xA7012948 & 0xFF)
        flag += chr(char_code)
        
        # Update gen for next character
        gen = ((gen ^ 0xA7012948 ^ char_code) + 131203) & 0xFFFFFFFF
    
    print(f"Flag: {flag}")

solve()
Flag
ENO{W0W_1_D1DN'T_533_TH4T_C0M1NG!!!}
