# 🚩 Neverland (Git Hook Privilege Escalation)

**Category:** System / Privilege Escalation
**Difficulty:** Medium
**Score:** 50

## Summary
The challenge required escalating from user `intern` to user `peter` by exploiting a flaw in the `sudo`-enabled `/opt/commit.sh` script, which failed to properly handle Git Hooks.

## The Flaw
1.  **Vulnerability:** The `intern` user was allowed to run `/opt/commit.sh` as `peter` via `sudo`.
2.  **The Hole:** The script ran `git commit` inside the user-controlled directory, which automatically executes any script found in the repository's `.git/hooks/` directory.
3.  **Logic Bug:** The script failed to validate the contents of the `.git/hooks/` folder. It also contained a directory traversal bug, making it check for `.git/config` inside the `.git` folder.

## The Solution
1.  **Bypass Checks:** Cloned the original `/app` repository and created a nested configuration (`.git/.git/config`) to pass the script's hash check.
2.  **RCE Payload:** Created a malicious shell script (`post-commit`) in the `.git/hooks/` directory with the content: `cat /home/peter/flag.txt > /tmp/flag_out`.
3.  **Execution:** Ran the script via `sudo -u peter /opt/commit.sh <payload.tar.gz>`.
4.  **Capture:** Read the flag from `/tmp/flag_out`.

**Flag:** Hero{c4r3full_w1th_g1t_hO0k5_d4dcefb250aa8c2ffabaa57119e3bc42}
