# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Fanl Escape** — single-file browser dodge game. No build step, no dependencies. Open `index.html` in a browser (serve locally via `python -m http.server` or `npx serve .` for `fetch()` to load txt files; built-in fallback tasks work even from `file://`).

## Files

```
index.html        — entire game (~760 lines: HTML + CSS + JS)
boss1.txt         — 30 falling task strings for Level 1 (Boss: Kuba)
boss2.txt         — 30 falling task strings for Level 2 (Boss: Vítek)
fanl_logo.png     — company logo (intro building display)
player1-3.png     — character photos (Maruška, Šárka, Mike)
boss1-2.png       — boss photos (Kuba, Vítek)
```

## Characters & Bosses

| | Name | Accusative | Image |
|---|---|---|---|
| Player 1 | Maruška | Marušku | player1.png |
| Player 2 | Šárka | Šárku | player2.png |
| Player 3 | Mike | Mika | player3.png |
| Boss 1 | Kuba | — | boss1.png |
| Boss 2 | Vítek | — | boss2.png |

## Architecture

All logic is in the `<script>` block of `index.html`:

| Section | Key identifiers |
|---|---|
| Config | `CHARS`, `BOSSES`, `FALLBACK`, `COLORS_COOL/MID/HOT` |
| State | `gs` object: `charIdx`, `played`, `rescued`, `level` |
| Screens | `show(id)` — swaps `.active` on `.screen` divs |
| Task loading | `loadTasks(lvl)` — `fetch()` + fallback to `FALLBACK` constant |
| Game loop | `loop(ts)` via `requestAnimationFrame` |
| Rendering | `draw(ts)` — bg → corridor walls → watermark → HUD → tasks → player |
| Flow | `selectCharacter` → `startLevel` → `winLevel`/`gameOver` → `continueAfterWin` → `afterBothLevels` → `showRescue`/`showFinalWin` |

## Game mechanics

- **Level duration:** 20 seconds per level
- **Play area:** centered corridor = 50% of screen width (same as progress bar), bounded by neon green walls; outside is darkened
- **Controls:** Arrow keys or A/D
- **Lives:** 3 hearts; 800 ms hit cooldown; screen shake on hit
- **Tasks:** spawn every ~1650 ms at start, accelerate to ~460 ms minimum; speed starts at 2.2, max 7
- **Color progression:** cool (blue/purple) → mid (orange/yellow) → hot (red) as time runs out
- **Text:** 20px bold Segoe UI, `textBaseline = 'middle'` for perfect centering in rectangles

## Rescue flow

`gs.rescued` is incremented in `doRescue` *before* the run starts:
- `rescued = 0` after initial run → "Unikl jsi z Fanlu!" + pick from 2 remaining chars
- `rescued = 1` after first rescue → "Zachránil jsi [acc]!" + pick last char
- `rescued = 2` after second rescue → final win screen

`gs.played` filters out already-used characters on rescue selection screens.

## Level win screen

- Label: "Úspěšně jsi unikl bossovi"
- Boss name shown in dative (`BOSSES[n].dative`)
- Subtitle after level 1: "PŘIPRAV SE NA BOSSE VÍTKA"
- Subtitle after level 2: "SKVĚLÁ PRÁCE!"
