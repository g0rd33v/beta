# BRICKS '86 — Product Specification

**Version:** 1.0
**Date:** 2026-08-01
**Author:** E. Gordeev / Claude
**Status:** Draft — awaiting approval

---

## 1. Vision & Overview

### 1.1 Problem Statement

In 1986, in Athens, the author played a simple paddle-ball-and-bricks game on an IBM
PC/XT-class clone — a machine with a separate monitor, a system box with 5.25″ floppy
drives, and a heavy clicky keyboard. The game was almost certainly **Bricks** (1984,
Vincent Bly), a freeware Breakout clone that spread on copied floppies, or a close
sibling of it. That experience — chunky text-mode bricks, a beeping PC speaker, a
keyboard-driven paddle — cannot be conveniently relived today without setting up a DOS
emulator and hunting down disk images.

### 1.2 Product Vision

A single, self-contained web page in the `g0rd33v/beta` repository that faithfully
recreates the *feel* of playing Bricks on a 1986 PC/XT: the same visual language
(text-mode CGA look, 16-color palette, blocky ball), the same sound language (square-wave
beeper tones), the same input language (keyboard-only paddle), and the same simplicity —
no power-ups, no enemies, no menus deeper than one screen. Opening the page should feel
like the machine just finished booting from a floppy.

### 1.3 Elevator Pitch

For anyone nostalgic for mid-80s PC gaming, **BRICKS '86** is a browser-playable
recreation of the 1984 freeware classic *Bricks* that reproduces the authentic
tweaked-CGA text-mode look and PC-speaker sound of an IBM PC/XT clone. Unlike modern
brick-breakers, it deliberately has no power-ups, no enemies, and no polish beyond what
1986 offered.

### 1.4 Success Metrics

1. The game is playable start-to-finish (title → game → game over → restart) with
   keyboard only.
2. The page is one HTML file with zero external network dependencies and loads instantly.
3. A player who used a real PC/XT recognizes the aesthetic without being told what it is.
4. Runs at a stable 60 fps in current Chrome/Firefox/Safari on desktop.

### 1.5 Scope & Phasing

| Feature                                        | Phase  | Priority |
|------------------------------------------------|--------|----------|
| Core paddle/ball/bricks gameplay               | v1     | Must     |
| CGA text-mode visual style + CP437-style font  | v1     | Must     |
| PC-speaker-style sound (WebAudio square waves) | v1     | Must     |
| Selectable color schemes (incl. mono phosphor) | v1     | Must     |
| Title screen, pause, game over, high score     | v1     | Must     |
| Boot sequence intro (cosmetic)                 | v1     | Should   |
| CRT scanline effect toggle                     | v1     | Should   |
| Touch/mouse paddle control (mobile fallback)   | v1     | Could    |
| "1989 mode" — Arkanoid-style lasers & aliens   | v2     | Deferred |
| Link from the landing page (`index.html`)      | v2     | Deferred |

Explicitly **out of scope for v1**: power-ups, enemies, lasers, multiple ball types,
level editors, online leaderboards, gamepad support, sound beyond the beeper.

---

## 2. Users & Personas

### 2.1 The Nostalgic Player (primary)

Played the original as a child in the 1980s. Wants the memory back: the look, the beeps,
the difficulty. Desktop browser, keyboard at hand. Will notice — and be bothered by —
anachronisms (smooth gradients, stereo sound effects, particle effects).

### 2.2 The Casual Visitor (secondary)

Lands on the page from the repo or a shared link. Has never seen a PC/XT. Needs the
controls to be discoverable in one glance (shown on the title screen) and the game to
work without reading anything else.

There are no admin users, accounts, or roles.

---

## 3. Information Architecture & Navigation

### 3.1 Screen Inventory

The entire app is one HTML page (`bricks.html`) that cycles through full-screen states:

| # | Screen          | Purpose                                                        |
|---|-----------------|----------------------------------------------------------------|
| 1 | Boot sequence   | Cosmetic ~3 s intro imitating a PC/XT floppy boot (skippable)  |
| 2 | Title screen    | Game name, controls, color-scheme selector, "press SPACE"      |
| 3 | Gameplay        | The playfield — paddle, ball, brick wall, status line          |
| 4 | Level cleared   | Brief interstitial, then next level                            |
| 5 | Game over       | Final score, session high score, "press SPACE to restart"      |

### 3.2 Navigation

Linear state machine driven by keyboard (see 5.3). No links, no scrolling, no menus.
The page renders as a centered 4:3 "monitor" area on a dark surround; everything happens
inside it.

### 3.3 Global Elements

A one-line status bar at the top of the playfield (text-mode style) is the only
persistent chrome: `SCORE 001240   HIGH 004890   BALLS ▮▮▮   LEVEL 2`.

---

## 4. Feature Specifications

### 4.1 Core Gameplay

#### 4.1.1 Description

Classic Breakout: the player moves a horizontal paddle along the bottom of the screen to
bounce a ball into a wall of bricks at the top. Each brick hit destroys the brick, scores
points, and reflects the ball. Losing the ball past the bottom edge costs one of a fixed
stock of balls. Clearing all bricks advances to the next, faster level.

#### 4.1.2 User Stories

- As a player, I want to move the paddle with the keyboard so that the game feels like
  the 1986 original.
- As a player, I want the ball's rebound angle to depend on where it strikes the paddle
  so that I can aim.
- As a player, I want each level to get faster so that a run stays challenging.

#### 4.1.3 Acceptance Criteria

1. Given the ball is in play, when it contacts the paddle's top surface, then it reflects
   upward with a horizontal angle determined by the contact offset (see rule 4.1.5.3).
2. Given the ball reaches the bottom edge, when no paddle contact occurred, then one ball
   is deducted, a "lost ball" tone plays, and the next serve waits for SPACE (or game
   over if the stock is empty).
3. Given the last brick of a level is destroyed, when the destroy animation/tone
   finishes, then the "level cleared" interstitial shows and the next level begins with
   increased ball speed.
4. Given any gameplay moment, when P is pressed, then all motion and sound stop until P
   is pressed again.

#### 4.1.4 Screen Walkthrough — Gameplay

- **Layout** (all measurements in text-mode character cells on a virtual 80×25 grid,
  rendered to canvas):
  - Row 0: status bar (score, high score, balls remaining, level).
  - Row 1: top wall — a solid line of block characters (`▓`).
  - Columns 0 and 79: side walls, same block character, rows 1–24.
  - Rows 4–11: the brick wall — 8 rows × 13 bricks. Each brick is 5 cells wide × 1 cell
    tall with a 1-cell gap column between bricks; rows are colored per the active scheme.
  - Row 23: the paddle — 8 cells wide, drawn as a solid block run (`▀` on `█`-style
    fill), moving in whole- or half-cell steps.
  - The ball: one character cell, drawn as `■` (a filled square, not a circle — CGA
    honesty).
  - Bottom edge (row 24 boundary): open — the ball exits here.
- **Actions**: ←/→ (and A/D) move paddle; SPACE serves the ball; P pauses; M mutes;
  C cycles color scheme; ESC returns to title (with confirmation via a second ESC).
- **Empty/loading states**: none — the game is fully client-side and instant.
- **Error state**: if WebAudio is unavailable, the game runs silently; a one-line
  `SOUND: OFF (NO SPEAKER)` notice shows in the status bar for 3 seconds.

#### 4.1.5 Business Rules (game rules)

1. **Lives**: the player starts each game with **5 balls** (matching the era's arcade
   convention; shown as `▮` glyphs).
2. **Scoring by row** (top row = row 1): rows 1–2 score **7** points per brick, rows 3–4
   score **5**, rows 5–6 score **3**, rows 7–8 score **1**. Example: clearing an entire
   level of 8×13 bricks scores (7+7+5+5+3+3+1+1)×13 = 416 points.
3. **Paddle rebound**: the paddle is divided into 8 contact zones; outermost zones
   reflect at the shallowest angle toward that side (≈30° from horizontal), center zones
   reflect steeply (≈70°). The ball's speed magnitude is unchanged by paddle hits.
4. **Speed-up rule** (classic Breakout): ball speed increases by one step after the 4th
   and 12th brick hit of each ball-in-play, and when a top-two-row (7-point) brick is
   first hit. Speed resets to the level's base speed on each new serve.
5. **Level progression**: each new level restores the full brick wall and raises the base
   ball speed by ~12%. There is no final level; the game is endless until balls run out.
   Levels are numbered from 1.
6. **Brick collision**: the ball destroys exactly one brick per contact frame and
   reflects on the axis of penetration (vertical hit → vertical bounce, side hit →
   horizontal bounce). No pass-through, no multi-brick smash.
7. **Serve**: after a serve prompt, the ball launches from the paddle's current position
   at a fixed 60° angle, direction (left/right) alternating per serve.
8. **High score**: the best score is kept in `localStorage` under key
   `bricks86.highscore` and shown on the status bar and game-over screen. If
   `localStorage` is unavailable, high score is session-only.

#### 4.1.6 Edge Cases

1. **Corner hits**: a ball striking the exact junction of two bricks destroys only the
   brick on the primary (vertical) axis.
2. **Paddle edge clip**: if the ball meets the paddle's vertical side while the paddle
   moves into it, reflect horizontally and upward (never trap the ball inside the
   paddle).
3. **Shallow-angle lock**: if the ball's vertical velocity magnitude would fall below a
   minimum (endless horizontal bouncing), it is nudged to the minimum vertical component.
4. **Tab loses focus / window blurred**: the game auto-pauses; a `PAUSED` overlay shows.
5. **Held keys on state change**: movement keys held during a state transition (e.g.,
   game over) must not carry into the next state (input state resets on transition).
6. **Double SPACE on serve**: serving is idempotent; extra presses within the same serve
   are ignored.

### 4.2 Authentic Presentation Layer

#### 4.2.1 Description

Everything the player sees and hears imitates a 1986 PC/XT clone with a CGA card and the
internal speaker.

#### 4.2.2 Visual rules

1. **Palette**: strictly the 16 CGA RGBI colors (`#000000`, `#0000AA`, `#00AA00`,
   `#00AAAA`, `#AA0000`, `#AA00AA`, `#AA5500`, `#AAAAAA`, `#555555`, `#5555FF`,
   `#55FF55`, `#55FFFF`, `#FF5555`, `#FF55FF`, `#FFFF55`, `#FFFFFF`). No other colors,
   no gradients, no alpha blending, no anti-aliasing (canvas smoothing off).
2. **Grid**: all rendering aligns to the 80×25 text-cell grid (each cell 8×16 virtual
   pixels → 640×400 virtual resolution, integer-scaled to fit the window while
   preserving 4:3).
3. **Typography**: all text is drawn in an embedded CP437-style 8×16 bitmap font
   (encoded as a data URI or drawn from an inline sprite — no external font files).
   Uppercase only, as the era demanded.
4. **Color schemes** (cycled with C, persisted in `localStorage` key `bricks86.scheme`):
   - `COLOR` — bricks in 8 row colors (red, yellow, magenta, green, cyan, blue, brown,
     light gray), white paddle/ball on black. Default.
   - `GREEN` — green-phosphor monochrome: everything in shades limited to green/bright
     green on black.
   - `AMBER` — amber-phosphor monochrome: brown/yellow on black.
   - `B&W` — light gray/white on black.
   In monochrome schemes brick rows are differentiated by glyph density (`░ ▒ ▓ █`),
   mirroring how the original stayed "quite playable in black and white."
5. **CRT effect**: optional overlay (toggle key T, default ON) adding faint horizontal
   scanlines and a subtle vignette. Pure cosmetics; no curvature warping of gameplay
   coordinates.

#### 4.2.3 Sound rules (PC speaker emulation)

1. All audio is generated with WebAudio using a **single square-wave oscillator** voice —
   the PC speaker could only play one tone at a time; a new sound cuts off the previous
   one. No volume envelopes beyond on/off; no stereo.
2. Event tones:
   | Event            | Tone                                |
   |------------------|-------------------------------------|
   | Paddle hit       | 220 Hz, 30 ms                       |
   | Wall hit         | 160 Hz, 20 ms                       |
   | Brick hit        | 400–1200 Hz (rises with brick row), 25 ms |
   | Serve            | two-note blip 440→880 Hz, 60 ms     |
   | Lost ball        | descending sweep 800→150 Hz, 350 ms |
   | Level cleared    | 4-note ascending arpeggio, 500 ms   |
   | Game over        | 3-note descending figure, 700 ms    |
3. Audio starts only after the first user gesture (browser autoplay policy); until then
   the game is silent and no error is shown.
4. M toggles mute; state persisted in `localStorage` key `bricks86.muted`.

### 4.3 Boot Sequence (cosmetic intro)

1. On page load, before the title screen, a ~3-second skippable sequence renders in
   text mode: memory count-up (`0064 KB OK` … `0640 KB OK`), a beep, a fake
   `Loading BRICKS.COM ...` line with floppy-seek clatter (short noise bursts), then a
   screen clear into the title.
2. Any key or click skips straight to the title screen.
3. The sequence plays only on page load, not on in-game restarts.

### 4.4 Title, Level-Clear, and Game-Over Screens

1. **Title**: game name in large block-character lettering, controls legend
   (`←/→ MOVE  SPACE SERVE  P PAUSE  M SOUND  C COLORS  T CRT`), current color scheme
   name, high score, and a blinking `PRESS SPACE TO PLAY`. C and T work here too, as a
   live preview.
2. **Level cleared**: playfield freezes, `LEVEL n CLEARED` centered for ~1.5 s with the
   arpeggio, then the next level auto-starts with a serve prompt.
3. **Game over**: `GAME OVER` centered over the dimmed playfield with final score and
   high score (with `NEW HIGH SCORE!` blink when applicable). SPACE returns to the title
   screen with everything reset.

---

## 5. Data Model

The game is fully client-side; "entities" are in-memory structures plus three persisted
keys.

### 5.1 In-memory state

| Entity   | Key attributes                                                        |
|----------|-----------------------------------------------------------------------|
| Game     | state (see 5.3), score, ballsRemaining, level, highScore              |
| Paddle   | x position (cells), width (8 cells), speed                            |
| Ball     | x, y (virtual px), vx, vy, speedStep, inPlay flag                     |
| BrickWall| 8×13 boolean grid + per-row color/score metadata; remainingCount      |
| Settings | scheme, muted, crtEffect                                              |

### 5.2 Persistence (`localStorage`)

| Key                 | Type   | Purpose                          |
|---------------------|--------|----------------------------------|
| `bricks86.highscore`| number | Best score across sessions       |
| `bricks86.scheme`   | string | Last selected color scheme       |
| `bricks86.muted`    | bool   | Sound on/off                     |

All reads are defensive: missing/corrupt values fall back to defaults (0, `COLOR`,
false).

### 5.3 Game state machine

```
[BOOT] --any key / 3 s--> [TITLE] --SPACE--> [SERVE] --SPACE--> [PLAYING]
[PLAYING] --ball lost, balls left--> [SERVE]
[PLAYING] --ball lost, no balls--> [GAME_OVER] --SPACE--> [TITLE]
[PLAYING] --last brick--> [LEVEL_CLEAR] --1.5 s--> [SERVE] (next level)
[PLAYING] <--P--> [PAUSED]
[SERVE|PLAYING] --ESC,ESC--> [TITLE]
```

| From        | Event               | To          | Side effects                        |
|-------------|---------------------|-------------|-------------------------------------|
| BOOT        | key/click or timer  | TITLE       | stop boot audio                     |
| TITLE       | SPACE               | SERVE       | reset score/balls/level/wall        |
| SERVE       | SPACE               | PLAYING     | launch ball, serve blip             |
| PLAYING     | ball exits bottom   | SERVE / GAME_OVER | decrement balls, lost-ball tone |
| PLAYING     | remainingCount = 0  | LEVEL_CLEAR | arpeggio, level++                   |
| LEVEL_CLEAR | 1.5 s timer         | SERVE       | rebuild wall, raise base speed      |
| PLAYING     | P / window blur     | PAUSED      | freeze physics, silence speaker     |
| PAUSED      | P / window focus+P  | PLAYING     | resume                              |
| GAME_OVER   | SPACE               | TITLE       | persist high score                  |

---

## 6. Business Logic Summary

Calculation and validation rules are fully covered in 4.1.5 (gameplay), 4.2.2 (visual),
and 4.2.3 (sound). There are no roles, permissions, server calls, or automations.

---

## 7. Integrations & External Systems

None. The page must make **zero network requests** beyond loading itself. The font and
any imagery are embedded in the single HTML file. This is a hard requirement (it also
keeps the page working when opened from a local file).

---

## 8. Non-Functional Requirements

### 8.1 Performance
- Fixed-timestep physics (e.g., 120 Hz simulation) decoupled from rendering via
  `requestAnimationFrame`; stable at 60 fps on a mid-range laptop.
- Page weight target: under 100 KB total.

### 8.2 Platforms
- Evergreen desktop Chrome, Firefox, Safari, Edge.
- Mobile: page renders and scales correctly; touch-drag paddle control is a
  nice-to-have (Could) — keyboard remains the primary input.

### 8.3 Accessibility
- Everything keyboard-operable (inherent to the design).
- `prefers-reduced-motion`: boot sequence auto-skips and CRT flicker effects (if any)
  stay static.
- Mute is one keypress and persistent.

### 8.4 Code & Repo
- One new file: `bricks.html` at the repo root (matching the existing flat structure of
  `g0rd33v/beta`); no build step, no dependencies, vanilla JS.
- This spec is committed alongside as `bricks-1986-specification.md`.
- Developed on branch `claude/1986-arkanoid-game-name-ybiwu4`.

---

## 9. Glossary

| Term        | Definition                                                            |
|-------------|-----------------------------------------------------------------------|
| PC/XT       | IBM PC (1981) / PC XT (1983) class machines and their clones — 8088 CPU, CGA graphics, internal beeper |
| CGA         | Color Graphics Adapter — 16-color text mode, 4-color graphics mode    |
| Tweaked CGA | Text-mode trick the original *Bricks* used to show more colors than CGA graphics mode allowed |
| CP437       | The original IBM PC character set (block glyphs `░▒▓█▮■` etc.)        |
| PC speaker  | The internal one-voice square-wave beeper of the PC                   |
| RGBI        | 4-bit color signal (red, green, blue, intensity) giving the 16 CGA colors |

---

## 10. Open Questions & Assumptions

### 10.1 Open Questions (for approval)

1. **Difficulty**: 5 starting balls and endless levels is proposed. Prefer 3 balls, or a
   fixed number of levels? — *Owner: E. Gordeev*
2. **File/page name**: `bricks.html` with in-game title **BRICKS '86** is proposed.
   Happy with the name? — *Owner: E. Gordeev*
3. **Landing page link**: should `index.html` get a link to the game now, or keep the
   game unlisted for v1? (Spec defers this to v2.) — *Owner: E. Gordeev*

### 10.2 Assumptions

1. Faithfulness to the *spirit* of 1986 outranks pixel-exact emulation of the original
   *Bricks* binary (whose exact layouts/keys varied by version).
2. Arrow keys (plus A/D) are acceptable substitutes for the original's Caps Lock/Ins
   paddle keys, which are impractical on modern keyboards.
3. The game ships as a static page in this repo; if the repo is served via GitHub Pages,
   it is playable at its URL with no further work.
