# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Score keeper for the card game Sâm Lốc: track players, enter per-round scores, view history, and compute who pays whom. It is a **single static file** — [index.html](index.html) contains all markup, CSS, and JavaScript (~1730 lines). There is no `package.json`, no build step, no test suite, and no local dependencies; every library is a CDN `<script>`/`<link>`.

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
| 70–530 | Layout, forms, buttons, player table, theme toggle, keyframes, `@media (max-width: 768px)` |
| 534–800 | History-modal styles (`.history-*`) |
| 805–930 | Dark-mode overrides for SweetAlert2 and Bootstrap classes |
| 934–1005 | Body markup — the entire static DOM |
| 1007 | jQuery loads here, *before* the app script; keep that order |
| 1008–1732 | The whole application, inside one `$(function () { ... })` closure |

## Data model

Two arrays live in the main closure and are the single source of truth:

```js
players = [{ name, score }]                                  // running totals
matches = [{ timestamp, players: [{ name, score }] }]         // per-round deltas
```

- **Totals are denormalized.** `players[i].score` is an accumulated sum, not derived from `matches` at render time. Any code that mutates `matches` must adjust `players[].score` itself — see `deleteMatchAt` ([index.html:1382](index.html#L1382)), which subtracts the round back out.
- **`name` is the join key** between the two arrays. Duplicate or renamed players corrupt totals. The history modal rebuilds its column list from *both* arrays (`recomputeTotals`, [index.html:1232](index.html#L1232)) so players no longer in the roster still show up.
- **Persistence** is `localStorage` under `players`, `matches`, `theme`. The convention throughout: mutate the array → `localStorage.setItem` → `renderTable()`. `renderTable()` ([index.html:1638](index.html#L1638)) is the only re-render path for the main table.

## Score entry semantics

Input is one comma-separated value per player, in roster order, with `-` marking the winner ([index.html:1127](index.html#L1127)). Numeric entries become *negative* points for that player; the `-` player receives `sumNumericValues(scores)` — the total the losers gave up — so a well-formed round sums to zero.

`validateInput` ([index.html:1719](index.html#L1719)) only checks the charset and that at least one `-` is present. Two `-` entries pass validation and each such player is credited the full loser sum, leaving a non-zero grand total; "Chia Tiền" then refuses to split. Keep that in mind before treating a zero-sum invariant as guaranteed.

## Money split

`#split-button` ([index.html:1445](index.html#L1445)) works on a copy of `players`, aborts unless the grand total is ≈ 0, then greedily pairs debtors with creditors, transferring `min(|debt|, credit)` per transaction.

## Share links

`#share-button` ([index.html:1557](index.html#L1557)) serializes `{players, matches}` to JSON, compresses with `LZString.compressToEncodedURIComponent`, and appends `?data=...` to `origin + pathname`. On load, `?data=` is decompressed and **overwrites localStorage without confirmation** ([index.html:1021](index.html#L1021)).

## SweetAlert2 is the entire modal layer

Every dialog — help, history, split results, confirmations, share — is a `Swal.fire` whose `html` is built by string concatenation. Consequences to respect:

- Modal DOM is re-created on each open, so handlers must be bound in `didOpen` and use `.off().on()` (`attachHandlers`, [index.html:1395](index.html#L1395)). Deleting a history row simply re-opens the modal to refresh it.
- Swal renders outside `.app-container`, so dark mode for dialog content requires explicit `body.dark-mode .swal2-*` / Bootstrap-class overrides in the 805–930 block. New dialog markup usually needs a matching override there.

## Theming

CSS custom properties on `:root`, re-declared under `body.dark-mode`; the toggle adds/removes the class on `<body>` and persists `theme`. A new color must be added to **both** blocks, and `applyTheme()`/`updateThemeIcon()` ([index.html:1620](index.html#L1620)) keep the icon in sync.

## Score animation

`renderTable` diffs `previousDisplayedScores` (keyed by player name) against the new totals and runs CountUp.js only on changed cells, guarded by `typeof countUp !== 'undefined'` because the CDN script may be unavailable. The map is display state only — never read it as data.

## Conventions

- All user-facing strings and code comments are **Vietnamese**; write new ones in Vietnamese too.
- The app code is jQuery-flavored ES5 (`var`, `function () {}`) with newer helpers using `const`/template literals. Match the surrounding style rather than modernizing wholesale.
- Commit messages follow conventional commits: `feat:`, `fix:`, `style:`, `refactor:`.
