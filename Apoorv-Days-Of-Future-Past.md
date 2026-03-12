# Days Of Future Past - ApoorvCTF Writeup

**Category:** Web / Crypto  
**Points:** 50  
**Author:** fl4nk3r  

## Challenge Description
CryptoVault - Secure Message Storage Platform. Can you get the secure message from the military grade security provided by our platform?

## Step 1: Reconnaissance (Information Disclosure)
Inspecting the HTML source code of the main page revealed several developer comments pointing to internal endpoints and leftover backup files. Fetching `/static/js/app.js` exposed a hardcoded path to a backup config file (`/backup/config.json.bak`) and mentioned a `/debug` endpoint that required an `X-API-Key`. Checking `/robots.txt` confirmed the existence of `/api/v1/debug` and an internal directory.

Downloading the backup config provided the necessary API key:
```bash
curl -s [http://chals1.apoorvctf.xyz:8001/backup/config.json.bak](http://chals1.apoorvctf.xyz:8001/backup/config.json.bak)
# Extracted Key: d3v3l0p3r_acc355_k3y_2024
```

## Step 2: Privilege Escalation (JWT Forgery)

Using the extracted API key, we queried the debug endpoint:

```
curl -s -H "X-API-Key: d3v3l0p3r_acc355_k3y_2024" [http://chals1.apoorvctf.xyz:8001/api/v1/debug](http://chals1.apoorvctf.xyz:8001/api/v1/debug)
```
The debug response leaked the entire authentication configuration for the application:
- Algorithm: HS256 (Symmetric encryption)
- Roles: viewer, editor, admin
- Target Endpoint: /api/v1/vault/messages (Requires admin role)
- Secret Derivation Hint: "Company name (lowercase) concatenated with founding year"

Following the hint ("cryptovault" + "2026"), the JWT signing secret was derived as cryptovault2026. With the algorithm and secret known, a forged JWT was crafted locally using Python's PyJWT library, elevating the role to admin.

Making a request to the vault endpoint with this forged token returned a JSON array containing 15 hex-encoded ciphertexts. The debug info noted the encryption method was an "XOR stream cipher".

## Step 3: Cryptography (Many-Time Pad Attack & The Human Element)

Because the developer used an XOR stream cipher but failed to implement a unique nonce/IV for each message, the exact same key stream was reused across all 15 ciphertexts. This is a classic Many-Time Pad vulnerability.
