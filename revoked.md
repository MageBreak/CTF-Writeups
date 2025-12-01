# HeroCTF: Revoked & Revoked Revenge
**Category:** Web / SQLi / Logic Flaw

## Analysis
Both challenges share the same source code with two critical flaws:
1.  **SQL Injection:** The `/employees` endpoint injects `query` directly into the SQL string, allowing us to leak data.
2.  **Revocation Bypass:** The database checks for revoked tokens using an exact match (`token = ?`), but the JWT library accepts tokens with extra padding (e.g., appended `=`).

## Solution
1.  **Leak Revoked Tokens:** Inject a UNION query to dump the `revoked_tokens` table.
    * Payload: `' UNION SELECT 1, token, 3, 4 FROM revoked_tokens --`
    * URL: `/employees?query=%27%20UNION%20SELECT%201%2C%20token%2C%203%2C%204%20FROM%20revoked_tokens%20--`

2.  **Steal Admin Token:** Decode the leaked tokens to find the one with `"is_admin": 1` (or the debug account for the first challenge).

3.  **Bypass Blacklist:** Append a single `=` to the end of the stolen token.
    * *Mechanism:* The database sees `Token=` (which is not in the blacklist), while the JWT decoder ignores the padding and accepts the valid signature.

4.  **Get Flag:** Replace your `JWT` cookie with the modified token (`Token=`) and navigate to `/admin`.
