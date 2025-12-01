# 🚩 Tomwhat (Cross-Context Session Pollution)

**Category:** Web / Configuration
**Difficulty:** Easy
**Score:** 50

## Summary
The goal was to bypass the username check in the `/dark/admin` application by manipulating the session attribute from a different, less-secure application.

## The Flaw
The `run.sh` setup script contained two critical misconfigurations:
1.  **Shared Cookies:** Set `sessionCookiePath="/"`, forcing all webapps to use the same session cookie.
2.  **Shared Storage:** Configured the `PersistentManager` to use the **same file directory** (`temp/sessions`) for both the secure (`/dark`) and insecure (`/light`) webapps.

## The Solution
1.  **Identify Weak Link:** The server was running the default Tomcat `/examples` application, which allows setting arbitrary session variables without validation.
2.  **Session Injection:** Accessed `/examples/servlets/servlet/SessionExample` and set the forbidden session attribute: `Name: username`, `Value: darth_sidious`.
3.  **Bypass:** Because the session storage was shared, the `/dark/admin` application loaded the malicious session file, found the required username, and printed the flag.

**Flag:** Hero{a2ae73558d29c6d438353e2680a90692}
