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

Addition/subtraction and multiplication/division carry separate number ranges, rather than sharing one. That lets each level push adding and subtracting further than multiplying and dividing — Level 3 adds numbers up to 9 but only multiplies up to 7 — so times tables can enter gently while the easier operations keep growing.

| Level | Name | +/− range | ×/÷ range | Add | Sub | Mult | Div |
|---|---|---|---|---|---|---|---|
| 1 | Warm-Up | 0–5 | — | 50% | 50% | — | — |
| 2 | Getting Going | 0–9 | 0–5 | 40% | 32% | 18% | 10% |
| 3 | Mixing It Up | 0–9 | 2–7 | 30% | 25% | 27% | 18% |
| 4 | Fact Power | 2–9 | 2–9 | 20% | 20% | 36% | 24% |
| 5 | Speed Demon | 3–9, − from up to 18 | 3–9 | 14% | 13% | 43% | 30% |
| 6 | Blitz Master | — | 4–9 | — | — | 59% | 41% |

**Level 6 drops adding and subtracting entirely.** It is pure times tables and division, drawn from factors of 4 through 9 — the corner of the table students find hardest. Level 5 is the last level where all four operations appear.

Subtraction always produces a non-negative answer, and division always divides evenly, so every answer is a whole number.

**10, 11 and 12 never appear as a number inside a question**, at any level. They are fine as answers, so `7 + 5 = 12` and `2 × 5 = 10` both occur, but `12 × 3` and `10 ÷ 2` do not. Drawn operands stop at 9 to handle most of this, though some numbers are computed rather than drawn — division dividends, and the teen minuends described below — so `NOT_IN_QUESTIONS` in the script catches leftovers like `12 ÷ 3 = 4`. There is no longer a separate rule for multiplying by 10, because 10 can no longer be a factor at all.

### Keeping + and − hard on Level 5

Level 5 is the top level that still adds and subtracts, and capping operands at 9 puts a low ceiling on how hard those can get. Two rules keep it on the difficult facts.

**Every sum crosses ten.** `minSum` in the `LEVELS` table rejects any addition under 10, which is the point where a student has to regroup rather than count. `7 + 8` stays, `4 + 5` is gone. Level 4 crosses ten about two thirds of the time; Level 5 always does.

**Subtraction borrows.** With both numbers under 10 there is nothing to borrow from, and the other constraints leave almost nothing to draw. So Level 5 usually takes the minuend from the teens instead, giving the borrowing facts that mirror the crossing-ten additions — `15 − 8`, `17 − 9`. Holding the answer below 10 is the same condition as requiring a borrow, which is why `18 − 4 = 14` never appears. `subTeen` sets how often this happens, currently about 60% of subtractions.

The remaining single-digit draws are governed by `minDiff`, which requires a difference of 3 or more so `9 − 7` stays out.

### No repeats within a sprint

**No question repeats within a sprint.** Each round tracks every problem shown and keeps drawing until it finds a fresh one.

Commutative pairs count as the same problem: if a player sees `3 + 7`, then neither `3 + 7` nor `7 + 3` will come back that round. Same for multiplication. Subtraction and division are already normalized by construction, so they need no special handling. The order shown is still randomized — you just never get both orders.

**Level 1 is the one exception, by design.** Numbers 0–5 with commutative pairs collapsed yield only 42 distinct problems: 21 additions and 21 subtractions. A fluent student answering faster than roughly one problem every 1.5 seconds will use them all up inside 60 seconds — in simulation the wall arrives around question 41, occasionally as early as 35. At that point the round starts a clean second pass rather than stalling, so nothing repeats until everything has been seen once, and the problem still on screen is excluded so a question never appears twice in a row.

**Level 6 is the other tight one**, at 57 distinct problems — factors of 4–9 give 21 multiplications and 36 divisions, and with adding and subtracting removed there is nothing else to draw on. That is only a risk for a very fast player: in simulation a 50-question round (1.2 seconds each) ran the pool dry in 0.4% of rounds, a 55-question round in 20% of them, and a 60-question round always. It resets the same way Level 1 does. The remaining pools are comfortable: Level 5 has 128 distinct problems and Levels 2 through 4 range from 152 to 166.

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
