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
- `Esc` ends a round early. The music-note button in the header mutes everything; it shows a slash through the note while sound is off.

## Difficulty levels

The slider picks one of six levels. Two things change as the level rises: the **operation mix** shifts from addition/subtraction toward multiplication/division, and the **numbers get bigger**.

Addition/subtraction and multiplication/division carry separate number ranges, rather than sharing one. That lets each level push adding and subtracting further than multiplying and dividing — Level 3 adds numbers up to 12 but only multiplies up to 7 — so times tables can enter gently while the easier operations keep growing.

| Level | Name | +/− range | ×/÷ range | Add | Sub | Mult | Div |
|---|---|---|---|---|---|---|---|
| 1 | Warm-Up | 0–5 | — | 50% | 50% | — | — |
| 2 | Getting Going | 0–10 | 0–5 | 40% | 32% | 18% | 10% |
| 3 | Mixing It Up | 0–12 | 0–7 | 30% | 25% | 27% | 18% |
| 4 | Fact Power | 2–12 | 1–9 | 20% | 20% | 36% | 24% |
| 5 | Speed Demon | 3–12 | 2–11 | 14% | 13% | 43% | 30% |
| 6 | Blitz Master | 4–12 | 2–12 | 8% | 7% | 50% | 35% |

Subtraction always produces a non-negative answer, and division always divides evenly, so every answer is a whole number.

**No question repeats within a sprint.** Each round tracks every problem shown and keeps drawing until it finds a fresh one.

Commutative pairs count as the same problem: if a player sees `3 + 7`, then neither `3 + 7` nor `7 + 3` will come back that round. Same for multiplication. Subtraction and division are already normalized by construction, so they need no special handling. The order shown is still randomized — you just never get both orders.

**Level 1 is the one exception, by design.** Numbers 0–5 with commutative pairs collapsed yield only 42 distinct problems: 21 additions and 21 subtractions. A fluent student answering faster than roughly one problem every 1.5 seconds will use them all up inside 60 seconds — in simulation the wall arrives around question 41, occasionally as early as 35. At that point the round starts a clean second pass rather than stalling, so nothing repeats until everything has been seen once, and the problem still on screen is excluded so a question never appears twice in a row. Levels 2 through 6 have pools of 178 to 277 problems and run well past 90 questions with no repeat at all.

Widening Level 1 or letting `3 + 2` and `2 + 3` both count would enlarge the pool, at the cost of the range or the no-repeat rule.

To retune the curriculum, edit the `LEVELS` object near the top of the `<script>` block — the weights (`w`) and both number ranges (`as` for add/subtract, `md` for multiply/divide) are all in one place.

## High scores

Each level keeps its own high score, stored in `localStorage` under `mathBlitzHighScores.v1`. Beating a level's record opens a classic arcade three-letter initials entry. Scores persist per browser, per device.

Clear them from the browser console:

```js
localStorage.removeItem('mathBlitzHighScores.v1')
```

## Audio

All sound is generated at runtime with the Web Audio API — the ding, the buzzer, the end-of-round fanfare, and the backing track (a 132 BPM I–V–vi–IV loop with bass, arpeggio, and drums). Browsers block audio until a user gesture, so sound starts when the player presses **START THE CLOCK**.
