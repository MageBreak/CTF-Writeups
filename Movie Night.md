# 🚩 Movie Night #1 (Privilege Escalation via SSH)

**Category:** System / Shell Escapes
**Difficulty:** Medium
**Score:** 50

## Summary
This was a classic shell challenge where the low-privileged user needed to find a specific binary or script they could execute as a higher-privileged user to read a flag located in the home directory of `peter`.

## The Flaw
The primary entry point was discovering that the user had permission to execute a specific script as the high-privileged user.

1.  **Discovery:** Running `sudo -l` revealed the user `intern` could execute `/opt/commit.sh` as `peter`.
2.  **Exploitation:** The script's lack of safety around the `.git/hooks` folder allowed the creation of a malicious `post-commit` script to execute arbitrary commands as the elevated user `peter`.

**Flag:** Hero{c4r3full_w1th_g1t_hO0k5_d4dcefb250aa8c2ffabaa57119e3bc42}
