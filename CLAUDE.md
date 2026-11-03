# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Gymmer is a personal, phone-first gym roadmap: a **single self-contained `index.html`** (~1800 lines) holding markup, CSS and JS. No build step, no dependencies, no package manager, no tests, no framework. Network calls are limited to YouTube thumbnails/embeds and the Google Fonts stylesheet; the page degrades to system fonts and stays fully usable offline.

The layout language is ported from the sibling project [Overlord Gym Tracker](https://github.com/strahinja0212-ctrl/Overlord-Gym-Tracker) — segmented tabs, hairline-grid tiles and mono labelling. **Themes are Gymmer's own:** six Warhammer 40k Space Marine chapters (personal, non-commercial use) — Ultramarines (default, bare `:root`), Blood Angels, Dark Angels, Space Wolves (light), Imperial Fists, Black Templars (light). Each chapter block declares base hues plus `--on-lift` (text on a `--lift` fill), `--trim` (heraldic second colour), `--sigil` and `--chapter` (masthead line). Sigils are single-colour SVG masks drawn for this page (`--sig-*` in the un-themed `:root`), tinted with `--lift`. The shared `/* chapter dressing */` block at the end of the `<style>` (masthead watermark + chapter line, h2 sigil marks, today-card watermark, sigil swatches) reads only tokens, so **a new chapter = one token block + one `--sig-*`** and a `THEMES` entry. Previous themes (Paper…Grimdark) are in `index.backup-pre-chapters.html`; `OLD_THEMES` in the JS maps saved old ids to chapters.

## Commands

```powershell
start index.html          # open in the default browser — that is the entire dev loop
```

Deployment is GitHub Pages from the repo root (see [README.md](README.md)). The working directory is not currently a git repository.

## Architecture

`index.html` is laid out in four blocks, in order:

1. **Theme tokens** — **six** complete color sets: `:root` (Ultramarines, the default, sets no attribute) plus `[data-theme="bloodangels"|"darkangels"|"spacewolves"|"imperialfists"|"blacktemplars"]`. Any new **color** must be added to **all six** or that theme breaks. Each theme declares only base hues (`--bg --surface --ink --muted --faint --line --line-strong --lift --lift-tint --run --run-tint --alt --good`); everything ordinal or categorical is *derived* from them with `color-mix` in the un-themed `:root` block below — the muscle ramp `--m1/--m2/--m3`, `--fig-base/--fig-line`, `--c-protein/--c-carb/--c-fat`, `--cal-on`. **Add a theme by declaring the base hues only** — never hand-pick ramp steps. The font/type/space/radius scale lives in that same un-themed block and must **not** be copied into a theme.
2. **Component CSS** — plain classes, no preprocessor. Three typefaces do specific jobs: **Cinzel** for anything that announces a section (`h1`–`h4`, `summary`, `.tile .v`), **Inter** for body copy, **JetBrains Mono** for every label, tag, table head and number. Utility classes (`.note`, `.row`, `.sw*`, `.full`, `.stack`) exist so the markup carries no `style=` attributes; the only three left are the macro bar's percentage widths, which are data.
3. **Markup** — a `.masthead` (eyebrow · display `h1` · lede) that scrolls away, then a sticky header holding the segmented `nav.tabs` and the `#themebar` swatches, then four `<section class="tab" id="tab-*">` panels: `workouts`, `nutrition`, `supps`, `tracker`. Below `<main>` sit `#rtimer` (the fixed rest-timer bar) and `<template id="bodyTpl">`, the front/back SVG body diagram cloned per workout *and* per exercise.
4. **`<script>`** — vanilla JS, `"use strict"`, no modules. Sections are delimited by `/* ═══ name ═══ */` banners.

At ≥960 px each `section.tab` becomes a two-column grid. `h2`, top-level `<p>` and anything marked `.full` span both columns; everything else pairs up. A card holding a wide table or the 26-week calendar needs `.full`.

### Content is data-attribute-driven

The JS reads declarative attributes out of the HTML rather than holding content in JS arrays. To add a workout or exercise, edit the markup; the behavior follows automatically.

- **`<tr data-vid="YOUTUBE_ID">`** — an exercise row. Its first cell carries a `.rowacts` div with two buttons, `form video` and `muscles`. Each opens a single shared `<tr class="exrow">` beneath it; opening one **replaces** the other, so a row can never stack two panels. `form video` shows a `hqdefault.jpg` thumbnail and only swaps in a `youtube-nocookie.com` iframe on the second click (a facade, to keep the page fast and cookie-free).
- **`EXMUS`** — the one deliberate exception to "content lives in the markup": a JS object mapping each `data-vid` to its muscle spec. The row still carries its identity declaratively; `EXMUS` is only the value store. It is keyed this way because 8 of the 36 rows repeat an exercise defined in another workout, and duplicated inline strings would silently drift the first time one is retuned. A per-row `data-mus="…"` overrides the table. Adding an exercise means adding a row *and* an `EXMUS` entry — a missing entry drops the `muscles` button, and an unknown muscle key logs a `console.warn`.
- **`<div class="muscles"></div>`** — the per-workout figure. It carries **no** `data-m`: the renderer walks its own `.body` table, looks each row up in `EXMUS`, and reduces with **`max`** ("the heaviest emphasis any lift here places on this muscle"), not sum — summing flattens every leg day to all-3s. Volume is reported separately as `(N sets)` in the text summary, counting only tier 2–3 work. Passing an explicit `data-m` still overrides the derivation.
- **`#bodyTpl`** — each figure is authored **once as its left half** and duplicated verbatim inside `<g transform="translate(200,0) scale(-1,1)">`, so no mirrored coordinate is ever computed by hand. The silhouette appears twice per half: closed (fill, no stroke) and open (stroke, no midline seam). Muscle keys must exist in the `MNAMES` map *and* as `data-m` shapes in the template; the renderer sets `i0`–`i3` (0 = untrained grey, 1–3 on the `--m1/--m2/--m3` ramp). There are 19 keys and every one is trained by something in the program — a permanently grey shape is worse than no shape.
- **Panes (cogitator terminal layout):** inside each `section.tab`, **every `<h2>` starts a pane**. At load the `/* ═══ panes ═══ */` JS wraps each h2 and everything up to the next h2 in a `.pane`, and builds a `.subnav` of numbered sub-tabs labelled by the h2's **`data-short`**. One pane shows at a time (remembered per tab in `gymmer.pane`, except Workouts, which always opens on Today). Each section must start with an h2, and every h2 needs a `data-short`. The h2 renders as the "SECTOR nn //" header bar (`data-n` is set by the JS). `openSession` calls `revealPane` before scrolling. Keys: `1`–`4` switch tabs, `←`/`→` step through panes. The desktop two-column grid applies to `.pane.on`, not the section. Keep copy terse: one short `.sub` line under a header at most, no filler intros.
- **`<section id="tab-NAME">`** — tab names are hardcoded in one place in `showTab`'s hash allowlist; adding a tab means touching the nav button, the section, and that array.

### State

All persistence is `localStorage` under the `gymmer.` prefix, via the `LS` get/set helper (JSON, swallows errors). Keys: `theme`, `tab`, `start` (ISO start date), `target` (3 or 4 sessions/week), `days` (`{"YYYY-MM-DD": 1}`), `lifts` (`{vid: [["YYYY-MM-DD", kg, reps], …]}`, oldest first, one entry per lift per day), `bw` (`{"YYYY-MM-DD": kg}`), `done` (`{day, ex:{"wo-id/vid": 1}}`, one day only), `lastBackup` (ISO date). The legacy `weights` key (`{name: {kg, date}}`, ten lifts) is migrated into `lifts` once by `migrateWeights` and then ignored.

- The **lift log** lists every unique `tr[data-vid]` in the programme (minus the timed holds in `NO_LOAD`), so adding an exercise row adds it here too. Reps `0` means "not recorded" (migrated entries). The sparkline plots Epley e1RM (`kg × (1 + reps/30)`), or reps alone for a lift never logged with load.
- **Day rollover:** `rollDay()` resets `done` when the date changes; it runs on every tick, and `onReturn` (visibilitychange/focus) also redraws the calendar, stats, today card and bodyweight date — a phone keeps the tab alive across midnight.

- The **today card** (`#todayCard`) is the one thing you look at before a session. Its schedule is **derived, never stored**: `start` gives the week, the weekday picks the slot (Phase 1 = Mon/Wed/Fri, Phases 2–3 = Mon/Tue/Thu/Fri via `P1_TRAIN`/`P2_TRAIN`), and Phase 1's A/B alternation falls out of counting the Mon/Wed/Fri slots already gone by — so `A-B-A` / `B-A-B` needs no state. Rest days offer the next session as "train anyway". `renderToday` also auto-opens that session's `<details>`, once, on load.
- The **rest timer** turns every Rest cell into a button. `restSeconds` takes the last number and the trailing unit ("60–90 s" → 90, "2–3 min" → 180, "90 s–2 min" → 120) — the upper bound, deliberately. The countdown is timestamp-based, not tick-counted, so a locked phone can't drift it; at zero it vibrates and beeps through WebAudio (no asset, stays offline).
- The **check-off** button in each `.rowacts` lives for a single day. The first tick of the day writes `days[today]`, so logging a session costs no extra taps. Its key is `detailsId + "/" + vid` because the same lift appears in several sessions.
- **Scroll targets:** `openSession` passes `keepScroll` to `showTab` so the two don't fight, and jumps instantly — a smooth scroll of several thousand pixels gets cancelled by the first stray touch.
- The **calendar** renders `WEEKS_BACK = 26` columns of 7 cells, Monday-first (`mondayOf`), one button per day; clicking toggles `days[key]` and re-runs `renderStats` (this week / week streak / total).
- Dates are keyed with the local-time `fmtDate` helper — never `toISOString()`, which would shift days across the timezone boundary.
- **Backup/restore** serializes `{v:3, start, target, days, lifts, bw}` to the clipboard, with a textarea fallback (and an honest "couldn't copy" message) when the clipboard API is unavailable. `lastBackup` is stamped only on a real copy or the "I copied it" button; `#bkAge` nags after 14 days. The page calls `navigator.storage.persist()` on load. Restore runs `checkBackup` over **every** field before writing any — one bad field rejects the whole paste — and converts v1/v2 `weights` via `migrateWeights`, then reloads. `done` is deliberately not backed up — it expires daily.

## Conventions

- Colors come only from the token variables, and never a raw hex outside a theme block. Check any new color in **all six** themes, including the two light ones (Space Wolves, Black Templars). Text on a `--lift` fill uses `--on-lift`, never `#fff`.
- **No emoji.** The design is entirely typographic; use a mono `.eyebrow`, a `.tag` chip or a Bricolage heading instead. This is the main thing that separates the current look from the old one.
- Prices, foods and shops are Serbia-specific (dinars, Maxi/Lidl, Ogistra/Proteini.si); nutrition targets assume a 70 → 90 kg bulk (~3,100 kcal, 150 g protein, +0.25–0.5 kg/week).
- Nutrition and supplement claims in the page are tied to the sources linked in the footer — keep new claims evidence-backed and cited there.
- Wide tables are wrapped in `<div class="scroll-x">`; the layout is mobile-first, 720 px until the 960 px breakpoint widens it.
- All interactive elements need a visible `:focus-visible` ring, and hover rules go inside `@media (hover: hover)` so they don't stick after a phone tap.
