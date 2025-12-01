# 🚩 The Chef's Secret Recipe

**Category:** Reverse Engineering (Dynamic Analysis)
**Difficulty:** Very Easy
**Score:** 50

## Summary
The flag was guarded by a simple string comparison function (`strcmp`) hidden within a Linux executable. The verbose "recipe" text was a distraction.

## The Flaw
The compiled executable took a single string argument and compared it directly against the true password/flag using the standard C library function `strcmp()`.

## The Solution
1.  **Dynamic Analysis:** Used the Linux tracing tool **`ltrace`** to intercept all library calls made by the executable.
2.  **Observation:** Ran the binary with a dummy flag (`ltrace ./file "Hero{test}"`).
3.  **Data Leak:** The `ltrace` output revealed the full hardcoded flag string in the `strcmp` arguments, which was previously truncated.

**Flag:** Hero{0h_N0_y0u_60T_My_S3cReT_C4k...}
