# Math Blitz

A 60-second math facts sprint for 3rd–6th graders. Single self-contained HTML file — no build step, no dependencies, no asset downloads.

## Run it

Open `index.html` in any modern browser. That's it.

Optionally serve it locally (useful if you want to test on a phone on the same network):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## How the game works

- The player has **60 seconds** to answer as many math facts as possible.
- Answers can be typed on the physical keyboard or the on-screen keypad (`DEL` to erase, `OK` to submit).
- A correct answer plays a rising **ding** and advances immediately. The ding pitches up as a streak builds.
- A wrong answer plays a **buzzer**, shakes the question, and lets the player try the same problem again — nothing is skipped.
- Upbeat instrumental background music (no lyrics) plays while the clock runs.
- `Esc` ends a round early. The `♪` button in the header mutes everything.

## Difficulty levels

The slider picks one of six levels. Two things change as the level rises: the **operation mix** shifts from addition/subtraction toward multiplication/division, and the **numbers get bigger**.

Addition/subtraction and multiplication/division carry separate number ranges. That lets Level 1 work with sums up to 9 while multiplication still enters gently at Level 2 with small factors, instead of forcing both onto one shared range.

| Level | Name | +/− range | ×/÷ range | Add | Sub | Mult | Div |
|---|---|---|---|---|---|---|---|
| 1 | Warm-Up | 0–9 | — | 50% | 50% | — | — |
| 2 | Getting Going | 0–10 | 0–5 | 40% | 32% | 18% | 10% |
| 3 | Mixing It Up | 0–12 | 0–7 | 30% | 25% | 27% | 18% |
| 4 | Fact Power | 2–12 | 1–9 | 20% | 20% | 36% | 24% |
| 5 | Speed Demon | 3–12 | 2–11 | 14% | 13% | 43% | 30% |
| 6 | Blitz Master | 4–12 | 2–12 | 8% | 7% | 50% | 35% |

Subtraction always produces a non-negative answer, and division always divides evenly, so every answer is a whole number.

**No question repeats within a sprint.** Each round tracks every problem shown and keeps drawing until it finds a fresh one.

Commutative pairs count as the same problem: if a player sees `3 + 7`, then neither `3 + 7` nor `7 + 3` will come back that round. Same for multiplication. Subtraction and division are already normalized by construction, so they need no special handling. The order shown is still randomized — you just never get both orders.

The smallest pool is Level 1 at 110 distinct problems, still well past what a fast player reaches in 60 seconds. If a pool ever did run dry, the round starts a clean pass rather than stalling.

To retune the curriculum, edit the `LEVELS` object near the top of the `<script>` block — the weights (`w`) and both number ranges (`as` for add/subtract, `md` for multiply/divide) are all in one place.

## High scores

Each level keeps its own high score, stored in `localStorage` under `mathBlitzHighScores.v1`. Beating a level's record opens a classic arcade three-letter initials entry. Scores persist per browser, per device.

Clear them from the browser console:

```js
localStorage.removeItem('mathBlitzHighScores.v1')
```

## Audio

All sound is generated at runtime with the Web Audio API — the ding, the buzzer, the end-of-round fanfare, and the backing track (a 132 BPM I–V–vi–IV loop with bass, arpeggio, and drums). Browsers block audio until a user gesture, so sound starts when the player presses **START THE CLOCK**.
