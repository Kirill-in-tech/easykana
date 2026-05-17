# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the site

No build step required — this is a pure static site. Serve locally with any HTTP server:

```bash
python3 -m http.server 8080
# or
npx serve .
```

Open `http://localhost:8080` in a browser. Files can also be opened directly as `file://` URLs, though audio playback may be restricted by browser security policies.

## Architecture

Five self-contained HTML pages with all JavaScript inline:

- `index.html` — Landing page with links to Hiragana and Katakana sections
- `intro.html` — Article explaining Japanese kana writing systems
- `hiragana.html` — Interactive character chart for Hiragana
- `katakana.html` — Interactive character chart for Katakana
- `quiz.html` — Multiple-choice quiz supporting Hiragana, Katakana, or both combined

`shared.css` provides the global design system (reset, nav, footer, theme variables, responsive breakpoints). Each page imports it and adds its own `<style>` block for page-specific styles.

`sounds/` contains MP3 files named by romaji syllable (e.g. `ka.mp3`, `shi.mp3`). All pages share one `<audio id="audioPlayer">` element whose `src` is swapped dynamically.

## Theming

Default theme is light. Dark mode is toggled by setting `data-theme="dark"` on `<html>` and persisted via `localStorage` key `"theme"`. The CSS uses two sets of custom properties: `:root` for light and `html[data-theme="dark"]` for dark. All color references use `var(--*)` tokens defined in `shared.css`.

## Character data

Each kana character is represented as a JS object inline in the page scripts:

```js
{ char: "あ", romaji: "a", sound: "./sounds/a.mp3", example: "..." }
```

`hiragana.html` and `katakana.html` each define their own array. `quiz.html` imports both arrays and merges them when "Both" mode is selected.

## Cache busting

`shared.css` is referenced with `?v=6`. Increment this version number when making changes to `shared.css` to force browser cache invalidation.

## Publishing

The site is hosted at `https://russet-guitar-j7p5.here.now/`. Publish via Claude Code CLI:

```bash
cd "/Users/kir/Projects/claude course/homework1/Final"
claude -p "publish this folder to https://russet-guitar-j7p5.here.now" --allowedTools "Bash"
```

## Navigation

Nav order: Intro | Hiragana | Katakana | Quiz. No "Home" link — the logo (`あ・EasyKana`) serves as the home link to `index.html`. The hamburger menu appears on mobile (≤768px).

## Row color palette

Characters are grouped by consonant row. Each row has a `--group-color` CSS variable set on `.group-block[data-group="..."]`. The colors cycle through 6 values and are used for the section title, vertical accent bar, and card romaji text.

Light mode:
- Vowels / M-row: `#0060d3` (blue)
- K-row / Y-row: `#6b21a8` (purple)
- S-row / R-row: `#c026d3` (pink)
- T-row / W-row: `#dc2626` (red)
- N-row: `#d97706` (amber)
- H-row: `#0891b2` (teal/cyan — originally green `#059669`, changed for palette cohesion)

Dark mode uses brighter variants of the same hues. The H-row dark mode color is `#50e3c2`.

**Important:** When clicking a card, the `:hover` state overrides `.card-romaji` color to `var(--text-primary)`. The detail panel reads `--group-color` from the parent `.group-block` via `getComputedStyle` to avoid this issue.

## Detail card styling

The character detail overlay (hiragana + katakana) has these styled elements:
- **Romaji**: bold, dynamically colored to match the character's row `--group-color`
- **Play Sound button**: 50% opacity gradient fill (`rgba(0,112,243,0.5)` → `rgba(121,40,202,0.5)` → `rgba(248,28,229,0.5)`), 75% on hover, white text/icon
- **Divider**: gradient line (`transparent` → blue → violet → pink → `transparent`)
- **"EXAMPLE WORD" label**: violet→pink gradient text

## Theme toggle

The toggle button matches the current theme (not contrasting). Base style: `background: var(--bg-secondary); color: var(--text-secondary); border: 1px solid var(--border)`. Hover brightens border and icon color. No gradient effects on the toggle.

## CSS transitions

Never use `transition: all` or `transition: border-color` — they cause visible delay during theme switches. Always use specific properties (e.g. `transform var(--transition), box-shadow var(--transition)`).

## Quiz features

- Script picker: Hiragana / Katakana / Both
- "Both" mode interleaves characters alternately (not random merge)
- Exit modal warns about progress loss when navigating away during an active quiz
- Exit modal intercepts all nav/logo clicks (no same-page exception)

## Audio playback

iOS Safari requires the audio element to be "unlocked" via a user gesture before `.play()` works. A `touchstart` listener (fired once) plays and immediately pauses the audio element to unlock it. This listener is **touch-only** — desktop browsers don't need it, and adding a `click` listener causes the first click's audio to fail due to a race condition with the actual `playSound()` call.

## Grid card play icons

Play icons (small circular button, top-right of each card) are **desktop-only**. Default `display: none`, switched to `display: flex` inside `@media (hover: hover)`. On hover the icon fades in; clicking it plays the sound without opening the detail card (`e.stopPropagation()`). On mobile, users tap the card to open the detail panel and use the Play Sound button there.

## Mobile considerations

- `touchend` with `e.preventDefault()` fixes Chrome mobile first-tap issues
- Touch scroll detection: touchstart records Y, touchend checks if moved >10px before triggering action
- `-webkit-tap-highlight-color: transparent` prevents card flickering
