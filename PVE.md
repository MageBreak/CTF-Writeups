# 🚩 PVE - Pirate Race #1 (Algorithmic Bot)

**Category:** Programming / Bot Warfare
**Difficulty:** Medium/Hard (Requires Coding)
**Score:** 257 (Dynamic)

## Summary
The goal was to write a Python function that uses the game state (island locations, barrel locations, ship position) to output control vectors (`acceleration`, `angle`) and achieve a superior score over a simple champion bot.

## The Flaw/Solution
The solution required overcoming two physics problems that sank early bots: **Overshooting** and **Crashing**.

1.  **Greedy Pathing:** The bot used a greedy algorithm, prioritizing Islands (to secure game progression) over Barrels.
2.  **Angle Physics:** Used the formula `math.atan2(dx, -dy)` to accurately map the coordinates to the game's "Clockwise from North" angle system (0°=North, 90°=East).
3.  **Braking Logic:** Implemented **Arrival Braking** (reduced throttle when distance was low) and **Turning Brake** (reduced throttle when `angle_diff` was high) to prevent the ship from overshooting and endlessly orbiting targets.

**Status:** Solved by winning 8/10 rounds (achieved 8/8) with the V3/V4 bot script, automatically submitting the flag.
