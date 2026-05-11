# Wordle 7

A 7-letter Wordle clone built as a Progressive Web App — no build tools, no framework, no backend. Fully offline-capable after first load.

Based on the original wordle-clone by [WebDevSimplified](https://github.com/WebDevSimplified/wordle-clone).

## How to Play

Guess the hidden 7-letter word in 6 tries. After each guess, tiles change color to show how close you were:

- **Green** — correct letter, correct position
- **Yellow** — correct letter, wrong position
- **Gray** — letter not in the word

A new word is available each day.

## Features

- 6×7 game board with animated tile reveals
- On-screen QWERTY keyboard with color feedback
- Hard mode — revealed hints must be used in subsequent guesses
- Share results as an emoji grid copied to clipboard
- Installable PWA — works offline via Service Worker cache
- Mobile-first responsive layout

## Running Locally

No build step required. Serve the files from any static file server:

```bash
npx serve .
# or
python3 -m http.server 8080
```

Then open `http://localhost:8080` in your browser.

> Opening `index.html` directly as a `file://` URL will work for basic gameplay but the Service Worker (offline support) requires an HTTP server.

## Project Structure

```
├── index.html        # Single-page shell: board, keyboard, modals
├── style.css         # All styles (mobile-first, CSS variables for theming)
├── app.js            # Game logic, DOM manipulation, state management
├── words.js          # Valid 7-letter answer words
├── manifest.json     # PWA metadata
├── sw.js             # Service Worker (Cache-First offline strategy)
└── icons/            # PWA icons (192px and 512px)
```

## Tech

Plain HTML5, CSS, and JavaScript. No dependencies, no bundler. Deployable to any static host (GitHub Pages, Netlify, Cloudflare Pages, etc.).
