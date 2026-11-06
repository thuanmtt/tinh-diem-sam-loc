# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Score keeper for the card game Sâm Lốc: track players, enter per-round scores, view history, and compute who pays whom. It is a **single static file** — [index.html](index.html) contains all markup, CSS, and JavaScript (~2100 lines). There is no `package.json`, no build step, no test suite, and no local dependencies; every library is a CDN `<script>`/`<link>`.

Deployed via GitHub Pages from the root of `master` → <https://thuanmtt.github.io/tinh-diem-sam-loc/>. Pushing to `master` publishes.

## Commands

```bash
open index.html                 # run it: file:// works, CDN libs load over the network
python3 -m http.server 8000     # use when you need a real origin (share links read window.location)
```

Verification is manual — load the page and exercise the flow you touched (add players → enter scores → history → chia tiền → share → dark mode).

## Map of index.html

| Lines | Contents |
|---|---|
| 8–25 | CDN links (Bootstrap 5, Font Awesome 6, SweetAlert2 11, LZString, CountUp.js) + Google Analytics `gtag` |
| 27–68 | `:root` design tokens (colors, gradients, radii, shadows, transition) |
| 54–68 | `body.dark-mode` token overrides |
| 70–432 | Layout, forms, buttons, player table, theme toggle |
| 433–680 | Round-entry sheet styles (`.round-*`): bottom sheet ≤ 600px, compact ≤ 360px |
| 681–780 | Keyframes, `@media (max-width: 768px)`, empty state |
| 781–1050 | History-modal styles (`.history-*`) |
| 1051–1178 | Dark-mode overrides for SweetAlert2 and Bootstrap classes |
| 1182–1243 | Body markup — the entire static DOM |
| 1244 | jQuery loads here, *before* the app script; keep that order |
| 1245–2108 | The whole application, inside one `$(function () { ... })` closure |

## Data model

Two arrays live in the main closure and are the single source of truth:

```js
players = [{ name, score }]                                  // running totals
matches = [{ timestamp, players: [{ name, score }] }]         // per-round deltas
```

- **Totals are denormalized.** `players[i].score` is an accumulated sum, not derived from `matches` at render time. Any code that mutates `matches` must adjust `players[].score` itself — see `deleteMatchAt` ([index.html:1771](index.html#L1771)), which subtracts the round back out.
- **`name` is the join key** between the two arrays. Duplicate or renamed players corrupt totals. The history modal rebuilds its column list from *both* arrays (`recomputeTotals`, [index.html:1621](index.html#L1621)) so players no longer in the roster still show up.
- **Persistence** is `localStorage` under `players`, `matches`, `theme`. The convention throughout: mutate the array → `localStorage.setItem` → `renderTable()`. `renderTable()` ([index.html:2027](index.html#L2027)) is the only re-render path for the main table.

## Score entry semantics

`#new-round-button` ([index.html:1528](index.html#L1528)) opens a Swal sheet with one row per player (bottom sheet on phones, centered dialog otherwise). The user taps a name to mark the single winner, then enters each loser's points (non-negative integers, `inputmode="numeric"`, blank = 0, ±1 steppers). `readRoundState` ([index.html:1436](index.html#L1436)) returns `{winnerIdx, losses, total}`; "Lưu ván" stays disabled until a winner is chosen and `total > 0`.

`saveRound` ([index.html:1505](index.html#L1505)) stores losers as `-loss` and credits the winner `total`, so every new round sums to zero. Rounds saved by the old comma-separated input could contain two winners and a non-zero total — "Chia Tiền" still refuses to split those, so don't assume the zero-sum invariant for old data.

Player names go into Swal HTML through `escapeHtml` ([index.html:1413](index.html#L1413)); the older dialogs (history, split) still concatenate names unescaped.

## Money split

`#split-button` ([index.html:1834](index.html#L1834)) works on a copy of `players`, aborts unless the grand total is ≈ 0, then greedily pairs debtors with creditors, transferring `min(|debt|, credit)` per transaction.

## Share links

`#share-button` ([index.html:1946](index.html#L1946)) serializes `{players, matches}` to JSON, compresses with `LZString.compressToEncodedURIComponent`, and appends `?data=...` to `origin + pathname`. On load, `?data=` is decompressed and **overwrites localStorage without confirmation** ([index.html:1260](index.html#L1260)).

## SweetAlert2 is the entire modal layer

Every dialog — round entry, history, split results, confirmations, share — is a `Swal.fire` whose `html` is built by string concatenation. Consequences to respect:

- Modal DOM is re-created on each open, so handlers must be bound in `didOpen` and use `.off().on()` (`attachHandlers`, [index.html:1784](index.html#L1784)). Deleting a history row simply re-opens the modal to refresh it.
- Swal renders outside `.app-container`, so dark mode for dialog content requires explicit `body.dark-mode .swal2-*` / Bootstrap-class overrides in the 1051–1178 block. The round-entry sheet needs none: it only uses CSS variables, which `body.dark-mode` already redefines. New dialog markup usually needs a matching override there.

## Theming

CSS custom properties on `:root`, re-declared under `body.dark-mode`; the toggle adds/removes the class on `<body>` and persists `theme`. A new color must be added to **both** blocks, and `applyTheme()`/`updateThemeIcon()` ([index.html:2009](index.html#L2009)) keep the icon in sync.

## Score animation

`renderTable` diffs `previousDisplayedScores` (keyed by player name) against the new totals and runs CountUp.js only on changed cells, guarded by `typeof countUp !== 'undefined'` because the CDN script may be unavailable. The map is display state only — never read it as data.

## Conventions

- All user-facing strings and code comments are **Vietnamese**; write new ones in Vietnamese too.
- The app code is jQuery-flavored ES5 (`var`, `function () {}`) with newer helpers using `const`/template literals. Match the surrounding style rather than modernizing wholesale.
- Commit messages follow conventional commits: `feat:`, `fix:`, `style:`, `refactor:`.
