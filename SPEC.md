# Wordle 7 — PWA Technical Specification

## Overview

A 7-letter Wordle clone built as a static HTML5 Progressive Web App. No build tools, no framework, no backend. Fully offline-capable after first load.

**Rules:** 6 guesses to find a 7-letter word. Tile colors reveal clues after each guess.

---

## File Structure

```
/
├── index.html          # Single-page shell: game board, keyboard, modals
├── style.css           # All styles (mobile-first, CSS variables for theming)
├── app.js              # Game logic, DOM manipulation, state management
├── words.js            # Exported array of valid 7-letter answer words
├── manifest.json       # PWA metadata (name, icons, theme color)
├── sw.js               # Service Worker for offline caching
└── icons/
    ├── icon-192.png    # PWA icon (Android home screen)
    └── icon-512.png    # PWA icon (splash screen)
```

---

## PWA Core

### `manifest.json`
Declares app name (`Wordle 7`), short name, start URL (`./`), display mode (`standalone` — hides browser chrome), background/theme color (`#ffffff`), and links to both icons. Used by browsers to offer "Add to Home Screen."

### `sw.js` — Service Worker
Uses a **Cache-First** strategy. On `install`, pre-caches all five core files (`index.html`, `style.css`, `app.js`, `words.js`, `manifest.json`). On `fetch`, checks the cache first and falls back to the network. Game works fully offline after the first load.

`index.html` registers the service worker on page load via a short inline `<script>` checking for `navigator.serviceWorker` support.

---

## Design & Layout (Mobile-First)

### Color Scheme — Classic (green/yellow/gray, light background)

CSS Custom Properties define all colors:

```
--color-correct:  #6aaa64  (green  — right letter, right position)
--color-present:  #c9b458  (yellow — right letter, wrong position)
--color-absent:   #787c7e  (gray   — letter not in word)
--color-empty:    #ffffff
--color-border:   #d3d6da
```

### Layout

Centered vertical flex column: `Header → Game Board → Keyboard`. Max-width capped at `500px` for desktop; base design targets 360–430px phone widths.

### Game Board

A 6×7 CSS Grid. Each tile is a square `<div>` sized with `clamp()` so tiles scale fluidly — large on desktop, compact on small phones without overflow.

### On-Screen Keyboard

Three rows of `<button>` elements in a QWERTY layout. Each key has a `data-key` attribute. Keys use `flex: 1` to fill rows evenly. Wide keys (`ENTER`, `⌫`) use `flex: 1.5`. Key background colors update as clues are revealed.

### Modals

Two overlay `<div>` elements (hidden by default):
- **Result modal** — win/loss message + Share Results button.
- **Settings panel** — gear icon in header opens it; contains the Hard Mode toggle.

---

## Core Logic Flow (`app.js`)

### 1. Initialization

On page load, derive today's answer: `words[dayIndex % words.length]` where `dayIndex` is computed from the calendar date. Load hard mode preference from `localStorage`.

### 2. State Object

A plain object tracks:

| Property | Type | Purpose |
|---|---|---|
| `currentRow` | number (0–5) | Which guess row is active |
| `currentCol` | number (0–6) | Which tile in the row is next |
| `currentGuess` | string | Letters typed so far |
| `gameOver` | boolean | Blocks further input |
| `revealedLetters` | Map\<string, status\> | Best known status per letter for keyboard coloring |
| `guessHistory` | string[] | Stored guesses for share emoji grid |

### 3. Input Handling

Both physical keyboard (`keydown`) and on-screen keyboard (`click`) funnel into one `handleKey(key)` function:

- **Letter key**: if `currentCol < 7` and game not over, append letter to `currentGuess`, fill tile at `[currentRow][currentCol]`, increment `currentCol`.
- **Backspace**: if `currentCol > 0`, remove last letter, clear tile, decrement `currentCol`.
- **Enter**: call `submitGuess()`.

### 4. Guess Submission (`submitGuess`)

1. Reject if `currentGuess.length < 7` — shake animation on the active row.
2. **Hard Mode check** (if enabled): verify all previously revealed green letters appear in their correct positions, and all yellow letters appear somewhere in the new guess. Reject with a toast message if violated.
3. **Scoring** — two-pass algorithm to handle duplicate letters correctly:
   - Pass 1: mark letters that are in the correct position as `correct` (green).
   - Pass 2: from the remaining unmatched letters, mark letters present elsewhere in the answer as `present` (yellow); all others are `absent` (gray).
4. **Reveal animation**: flip each tile with a staggered CSS `animation-delay` of 100ms per tile, applying `.correct`, `.present`, or `.absent` classes on flip.
5. **Update keyboard**: after each tile reveals, upgrade the corresponding key's color class — status only ever upgrades (absent → present → correct, never downgraded).
6. **Check win/loss**: after the final tile reveals, if guess === answer (win) or `currentRow === 5` (loss), set `gameOver = true` and show the result modal after a short delay.

### 5. Share Results Button

Builds an emoji grid string (🟩🟨⬛) from `guessHistory` and calls `navigator.clipboard.writeText()`. Falls back to `document.execCommand('copy')` for older browsers. Shows a "Copied!" toast on success.

### 6. Hard Mode Toggle

Checkbox in the settings modal. Can only be toggled before the first guess of a game. State saved to and read from `localStorage`.

---

## Key Technical Decisions

| Decision | Rationale |
|---|---|
| No build tools / no framework | Zero dependencies, instant load, deployable to any static host |
| CSS Grid for board | Precise tile sizing without JS; scales with `clamp()` |
| Two-pass scoring for yellow tiles | Correctly handles words with repeated letters (e.g. `STREETS`) |
| `localStorage` for hard mode pref | Persists across sessions without a backend |
| Staggered CSS animation via `animation-delay` | Smooth tile reveal without JS timers |
| Bundled word list (`words.js`) | Fully offline; no network request needed for answers |

---

## Features Included

- [x] 6 guesses for a 7-letter word
- [x] On-screen QWERTY keyboard with color feedback
- [x] Classic color scheme (green / yellow / gray, light background)
- [x] Share results button (emoji grid copied to clipboard)
- [x] Hard mode toggle (persisted to localStorage)
- [x] Fully offline via Service Worker cache
- [x] Mobile-first responsive layout
