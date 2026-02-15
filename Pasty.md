# NullCon CTF 2025: Pasty Write-up

**Category:** Web / Crypto  
**Author:** @gehaxelt  
**Difficulty:** Medium  

---

## Challenge Description

> "Check out our new secure pastebin service! We rolled our own cryptographic signatures to protect paste access - after all, why trust those boring standard libraries when you can build something custom? Can you prove that our homebrewed crypto isn't as secure as we think and get access to the 'flag' paste?"

We are provided with the source code (`sig.php`) and a target website. The goal is to access a restricted paste with the ID `flag`.

---

## Source Code Analysis

The core logic lies in `sig.php`, specifically the `compute_sig` function.

```php
function _x($a, $b) {
    $r = '';
    for ($i = 0; $i < strlen($a); $i++) $r .= chr(ord($a[$i]) ^ ord($b[$i]));
    return $r;
}

function compute_sig($d, $k) {
    $h = hash('sha256', $d, 1);
    $m = substr(hash('sha256', $k, 1), 0, 24); // Master Key Stream (24 bytes)
    $o = '';
    
    for ($i = 0; $i < 4; $i++) {
        $s = $i << 3; 
        $b = substr($h, $s, 8); // Current 8-byte block of the input hash
        
        // VULNERABILITY HERE:
        $p = (ord($h[$s]) % 3) << 3; 
        $c = substr($m, $p, 8); // Selects 8 bytes from $m based on input hash
        
        // CBC-like XOR construction
        $o .= ($i ? _x(_x($b, $c), substr($o, $s - 8, 8)) : _x($b, $c));
    }
    return $o;
}
```

---

## The Vulnerability: Key Reuse & Weak Selection

The custom "MAC" (Message Authentication Code) algorithm has two fatal flaws:

### 1. Deterministic Key Selection

The 8-byte key chunk `$c` used for each block is chosen from the master key `$m`.

Crucially, the selection index:

```
$p = (ord($h[$s]) % 3) << 3;
```

This means there are only **3 possible key chunks** (`K_0`, `K_1`, `K_2`) used for the entire operation.

The choice of which key to use is determined by the input data itself (specifically, the first byte of each block of the input's SHA256 hash).

---

### 2. Reversible XOR Operations

The signature construction is essentially:

```
Block 0: Sig_0 = Hash_0 ⊕ K_idx
Block n: Sig_n = Hash_n ⊕ K_idx ⊕ Sig_{n-1}
```

Because XOR is reversible:

```
A ⊕ B = C  ⇒  C ⊕ B = A
```

If we know the input hash (`Hash_0`) and the resulting signature (`Sig_0`), we can trivially recover the key (`K_idx`).

---

## Attack Strategy (Chosen Plaintext Attack)

To get the flag, we need to forge a signature for `id=flag`.

We don't know the master key `$m`, but we can recover the 3 key chunks (`K_0`, `K_1`, `K_2`) by using the server as an oracle.

### Step-by-step Plan

1. **Generate Test Data**
   - Create new pastes on the server.
   - The server generates a random ID for each paste and signs that ID.

2. **Map IDs to Keys**
   - Calculate the SHA256 hash of these IDs locally.
   - Look at the first byte of the hash modulo 3.
   - This tells us which key chunk (`K_0`, `K_1`, or `K_2`) the server used to sign the first block.

3. **Recover Keys**
   - Find an ID where the first block uses `K_0`.
   - Use the formula:
     ```
     K_0 = Sig_0 ⊕ Hash(ID)_0
     ```
   - Repeat for `K_1` and `K_2`.

4. **Forge**
   - Once we have all 3 key chunks, we can simulate the `compute_sig` function locally.
   - Generate a valid signature for the ID `flag`.

---

## Solution Scripts

### Step 1: Finding Valid IDs

We first needed to find random IDs on the server that mapped to key indices 0, 1, and 2.  
We created pastes, grabbed the IDs/signatures, and ran this check:

```python
import hashlib

# IDs and Signatures harvested from the challenge server
candidates = [
    ("700e8a308cde5a81", "e2aa3e471ec66e2b42fa22384e6f9a10a0378b8bffee25321857f440642a6bfc"),
    ("e95f582eed8c686c", "584bc69f62a01d97586a9774cba22b73161866d0f890e1c8ed6a135656c56ba8"),
    ("bc8ce466a573ec9a", "49c783605b6d7f62e9c831439fde7baa0b8ac70c91c92797240dc6c99d087cc6")
]

for id_val, sig in candidates:
    h = hashlib.sha256(id_val.encode()).digest()
    key_idx = h[0] % 3
    print(f"ID {id_val} uses Key Index: {key_idx}")
```

---

### Step 2: Key Recovery & Forgery

With valid IDs for indices 0, 1, and 2, we wrote `solve.py` to recover the keys and forge the flag signature.

```python
import hashlib
import binascii

# --- DATA HARVESTED FROM SERVER ---

# Key Index 0 ID/Sig
id_k0 = "700e8a308cde5a81"
sig_k0 = "e2aa3e471ec66e2b42fa22384e6f9a10a0378b8bffee25321857f440642a6bfc"

# Key Index 1 ID/Sig
id_k1 = "e95f582eed8c686c"
sig_k1 = "584bc69f62a01d97586a9774cba22b73161866d0f890e1c8ed6a135656c56ba8"

# Key Index 2 ID/Sig
id_k2 = "bc8ce466a573ec9a"
sig_k2 = "49c783605b6d7f62e9c831439fde7baa0b8ac70c91c92797240dc6c99d087cc6"

def xor_bytes(a, b):
    return bytes(x ^ y for x, y in zip(a, b))

def get_first_block_key(id_string, signature_hex):
    # 1. Decode Signature
    sig_bin = binascii.unhexlify(signature_hex)
    o_0 = sig_bin[0:8]  # First 8 bytes of output
    
    # 2. Hash the ID
    h = hashlib.sha256(id_string.encode()).digest()
    b_0 = h[0:8]  # First 8 bytes of hash
    
    # 3. Recover Key: K = Sig ^ Hash
    return xor_bytes(o_0, b_0)

print("[*] Recovering Key Chunks...")
K0 = get_first_block_key(id_k0, sig_k0)
K1 = get_first_block_key(id_k1, sig_k1)
K2 = get_first_block_key(id_k2, sig_k2)

print(f"Key 0: {binascii.hexlify(K0).decode()}")
print(f"Key 1: {binascii.hexlify(K1).decode()}")
print(f"Key 2: {binascii.hexlify(K2).decode()}")

print("\n[*] Forging signature for ID 'flag'...")

# Target ID
target = "flag"
h_target = hashlib.sha256(target.encode()).digest()
blocks = [h_target[i:i+8] for i in range(0, 32, 8)]

# The key sequence for 'flag' is determined by its hash bytes % 3
# In this specific case, 'flag' maps to indices: [2, 0, 2, 1]
keys = [K2, K0, K2, K1]

final_sig = b""
prev_block = b""

for i in range(4):
    b_curr = blocks[i]
    k_curr = keys[i]
    
    # Calculate: Hash ^ Key
    val = xor_bytes(b_curr, k_curr)
    
    # CBC Chaining: XOR with previous block (except for the first one)
    if i > 0:
        val = xor_bytes(val, prev_block)
        
    final_sig += val
    prev_block = val 

forged_sig = binascii.hexlify(final_sig).decode()
print(f"\nSUCCESS! FORGED SIGNATURE: {forged_sig}")
print(f"URL: http://52.59.124.14:5005/view.php?id=flag&sig={forged_sig}")
```

---

## Result

Running the script produced the valid signature for the ID `flag`:

```
SUCCESS! FORGED SIGNATURE: b8e4e53e526806aa641e0dac06294218f67a1e18ea7c2d573d40a266a7703710
```

Accessing the URL revealed the flag:

```
ENO{cr3at1v3_cr7pt0_c0nstruct5_cr4sh_c4rd5}
```
