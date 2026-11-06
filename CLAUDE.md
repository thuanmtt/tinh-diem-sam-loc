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
| 433–610 | Round-entry wizard styles (`.round-*`); on ≤ 600px the popup sits at the top so the keyboard never covers it |
| 611–710 | Keyframes, `@media (max-width: 768px)`, empty state |
| 711–980 | History-modal styles (`.history-*`) |
| 981–1108 | Dark-mode overrides for SweetAlert2 and Bootstrap classes |
| 1112–1173 | Body markup — the entire static DOM |
| 1174 | jQuery loads here, *before* the app script; keep that order |
| 1175–2064 | The whole application, inside one `$(function () { ... })` closure |

## Data model

Two arrays live in the main closure and are the single source of truth:

```js
players = [{ name, score }]                                  // running totals
matches = [{ timestamp, players: [{ name, score }] }]         // per-round deltas
```

- **Totals are denormalized.** `players[i].score` is an accumulated sum, not derived from `matches` at render time. Any code that mutates `matches` must adjust `players[].score` itself — see `deleteMatchAt` ([index.html:1727](index.html#L1727)), which subtracts the round back out.
- **`name` is the join key** between the two arrays. Duplicate or renamed players corrupt totals. The history modal rebuilds its column list from *both* arrays (`recomputeTotals`, [index.html:1577](index.html#L1577)) so players no longer in the roster still show up.
- **Persistence** is `localStorage` under `players`, `matches`, `theme`. The convention throughout: mutate the array → `localStorage.setItem` → `renderTable()`. `renderTable()` ([index.html:1983](index.html#L1983)) is the only re-render path for the main table.

## Score entry semantics

`#new-round-button` ([index.html:1502](index.html#L1502)) opens one Swal popup that runs a two-step wizard, all inside `bindRoundWizard` ([index.html:1403](index.html#L1403)): first "Ai thắng ván này?" (one button per player), then one screen per loser in roster order — a single numeric input (`inputmode="numeric"`, blank = 0), "Lưu" moves to the next loser, the last one shows "Lưu ván". Chips let the user jump to any loser; "Quay lại" steps back (to the winner screen from the first loser). The input element is reused across steps and buttons `preventDefault` on `mousedown`, so the phone keyboard stays open; focus is called inside the tap handler so iOS opens it.

Wizard state is `{winnerIdx, pos, losses}` (`losses` = raw strings per player index), replaced with `Object.assign` on every step. Closing the popup discards the round. The last step refuses a round whose loser total is 0.

`saveRound` ([index.html:1478](index.html#L1478)) stores losers as `-loss` and credits the winner `total`, so every new round sums to zero. Rounds saved by the old comma-separated input could contain two winners and a non-zero total — "Chia Tiền" still refuses to split those, so don't assume the zero-sum invariant for old data.

Player names go into Swal HTML through `escapeHtml` ([index.html:1343](index.html#L1343)); the older dialogs (history, split) still concatenate names unescaped.

## Money split

`#split-button` ([index.html:1790](index.html#L1790)) works on a copy of `players`, aborts unless the grand total is ≈ 0, then greedily pairs debtors with creditors, transferring `min(|debt|, credit)` per transaction.

## Share links

`#share-button` ([index.html:1902](index.html#L1902)) serializes `{players, matches}` to JSON, compresses with `LZString.compressToEncodedURIComponent`, and appends `?data=...` to `origin + pathname`. On load, `?data=` is decompressed and **overwrites localStorage without confirmation** ([index.html:1190](index.html#L1190)).

## SweetAlert2 is the entire modal layer

Every dialog — round entry, history, split results, confirmations, share — is a `Swal.fire` whose `html` is built by string concatenation. Consequences to respect:

- Modal DOM is re-created on each open, so handlers must be bound in `didOpen` and use `.off().on()` (`attachHandlers`, [index.html:1740](index.html#L1740)). Deleting a history row simply re-opens the modal to refresh it.
- Swal renders outside `.app-container`, so dark mode for dialog content requires explicit `body.dark-mode .swal2-*` / Bootstrap-class overrides in the 981–1108 block. The round-entry wizard needs none: it only uses CSS variables, which `body.dark-mode` already redefines. New dialog markup usually needs a matching override there.

## Theming

CSS custom properties on `:root`, re-declared under `body.dark-mode`; the toggle adds/removes the class on `<body>` and persists `theme`. A new color must be added to **both** blocks, and `applyTheme()`/`updateThemeIcon()` ([index.html:1965](index.html#L1965)) keep the icon in sync.

## Score animation

`renderTable` diffs `previousDisplayedScores` (keyed by player name) against the new totals and runs CountUp.js only on changed cells, guarded by `typeof countUp !== 'undefined'` because the CDN script may be unavailable. The map is display state only — never read it as data.

## Conventions

- All user-facing strings and code comments are **Vietnamese**; write new ones in Vietnamese too.
- The app code is jQuery-flavored ES5 (`var`, `function () {}`) with newer helpers using `const`/template literals. Match the surrounding style rather than modernizing wholesale.
- Commit messages follow conventional commits: `feat:`, `fix:`, `style:`, `refactor:`.
