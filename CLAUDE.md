# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Coffee Pinball is a single-file web game (~2900 lines) where players drop coffee-themed balls through a pinball board into cups to decide who buys coffee. It supports both solo (offline) and multiplayer (online via Firebase Realtime Database) modes. Deployed as a static page on GitHub Pages at `yunuuuk1.github.io/coffee-pinball/`.

## Development

There is no build system, bundler, or package manager. The entire app is a single `coffee-pinball.html` file containing inline CSS, HTML, and JavaScript. A `.bak` file exists as a manual backup.

- **To run locally**: Open `coffee-pinball.html` in a browser. Multiplayer features require Firebase connectivity.
- **No tests, linting, or CI** exist in this project.

## Architecture

### Single-File Structure (coffee-pinball.html)

The file is organized in this order:

1. **CSS styles** (lines 30–211): All styling is inline in `<style>`. Class names are heavily abbreviated (e.g., `.pr` = player row, `.sb` = settings box, `.gb` = go button, `.rr` = ranking row).
2. **HTML markup** (lines 213–439): UI screens are toggled via `display:none/block`. Key screen IDs:
   - `mainMenu` — Main menu with Solo/Create/Join/History buttons
   - `createScreen`, `joinScreen`, `invitePage` — Multiplayer room setup
   - `lobby` — Waiting room with QR code, player list, chat log
   - `setup` — Solo mode player name entry and game settings
   - `game` — Canvas-based pinball game area with ranking sidebar
   - `rs` — Single-round result screen
   - `roundSum` — Multi-round final summary
   - `histPanel` — Game history with room tabs, date filters, charts
3. **JavaScript** (lines 441–2907): All game logic in a single `<script>` block.

### JavaScript Sections (marked with `═══` comment headers)

- **Constants & State** (~450–477): `CANVAS_W=500, CANVAS_H=750`, physics constants, player/ball/cup arrays, game state flags.
- **Audio** (~479–504): Web Audio API tone synthesis for sound effects and BGM. No audio files — all procedural.
- **Setup** (~506–512): Player input management, validation.
- **Ranking** (~514–518): Real-time leaderboard bar chart rendered in HTML.
- **Cheer/Toast/Particles/Confetti** (~520–653): Visual effects systems using canvas drawing.
- **Shared Rendering** (~555–653): Reusable canvas draw functions (background, vignette, balls, cups, spotlight) shared between host and spectator views.
- **Dramatic Reveal** (~655–780): Multi-phase winner announcement animation (spotlight fade → confetti → result UI).
- **Game Init & Flow** (~780–1000): `initB()` builds obstacle grid and cups. `startRound()`, `countdown()`, `animate()`, `startDropTimer()` control game lifecycle.
- **Final Bomb** (~1000–1055): End-game mega ball mechanic with staggered spawn timing.
- **Collision & Physics** (~1057–1163): Polygon collision detection, ball-pin bouncing, funnel wall physics, cup scoring. Supports circle/triangle/diamond/rect/hexagon obstacles.
- **Draw** (~1165–1230): Main render loop drawing obstacles, balls, cups, particles, overlays.
- **Result & Tiebreaker** (~1230–1340): Score display, multi-round cumulative scoring, tiebreaker logic (up to 3 rounds).
- **Multiplayer Room System** (~1342–1700): Firebase-based room create/join/lobby, real-time player sync, QR code generation, invite links with `?room=` URL param.
- **Spectator System** (~1700–2175): Non-host players watch the game via Firebase state sync. Has its own render loop (`startSpecCanvas`) with ball position interpolation and all visual effects mirrored from host.
- **Cheer System** (~2177–2500): Multiplayer emoji reactions with particle effects on a separate full-screen canvas.
- **Invite Flow & Firebase Init** (~2500–2530): URL param detection for invite links, async Firebase SDK loading.
- **History & Persistence** (~2530–2905): Game records saved to Firebase + localStorage fallback. Filterable by room and date range. Chart.js integration for stats visualization.

### Key Design Patterns

- **Screen navigation**: All via `display:none/block` toggling of div IDs. `userNav` flag prevents auto-show of mainMenu after user navigates.
- **Canvas rendering**: Single `<canvas id="cv">` at 500x750, scaled to container width. Host runs physics + rendering; spectators run interpolated rendering from Firebase state.
- **DOM helper**: `const $=id=>document.getElementById(id)` used throughout.
- **Special ball types**: `normal`, `gold` (-3 score), `bomb` (+3), `megagold` (-5 + splash), `megabomb` (+5 + splash). Mega balls have larger collision radius and ultra-slow gravity.
- **Pin types**: `normal`, `bumper` (2x velocity), `slow` (0.33x velocity), `redirect` (random angle). Each has distinct colors.
- **Firebase structure**: `rooms/{code}/` for multiplayer state, `games/` for history records. Host pushes `gameState` updates; spectators read them.

### External Dependencies (loaded via CDN)

- Google Fonts: Noto Sans KR
- Firebase 10.12.0 (app-compat + database-compat) — loaded async
- Chart.js 4.4.0 — loaded async, used only in history panel
- QR Server API (`api.qrserver.com`) — for QR code generation

## Language

UI text is primarily in English with some Korean labels (day names, button text). The HTML lang is set to `ko`.
