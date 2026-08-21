# WPA-National-Park-Vintage

A design system for printed-badge UI: 1930s–1970s National Park Service "Unigrid"
layout rules, WPA poster construction, flat offset block shadows — carrying the
Jambo Leaderboard's own slate-blue-and-gold shore palette rather than a park
green. Built for the Jambo Leaderboard and intended to move to Claude Design.

**Open `design-system/index.html` in a browser** — that page is the visual spec.
Every component renders there, styled entirely by `wpa.css`.

---

## What's here

```
wpa.css      the entire system — tokens and components. Single source of truth.
fonts/       National Park, latin subset, weights 400/700/800 (~47 KB total)
index.html   the styleguide
README.md    this file
```

`wpa.css` is linked by both the leaderboard (`../index.html`) and the styleguide,
so the two cannot drift. There is no build step and no dependencies.

## Consuming it

```html
<link href="https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&family=Yellowtail&display=swap" rel="stylesheet">
<link rel="stylesheet" href="design-system/wpa.css">
```

Three faces: **Yellowtail** for the wordmark only (`--font-script`), **National Park**
self-hosted for all display type (`--font-display`), **Courier Prime** for body and
score columns (`--font-body`).

`body` takes `--page-ground` (sky), not `--background`. The **board is the paper** —
cards, badges and the board itself take `--background`. This split exists because the
illustration sits behind the whole page.

`wpa.css` sets `html, body { overflow: hidden; height: 100% }` because the
leaderboard is a locked one-screen board. Any other page opts out with
`<html data-doc>`, which restores normal scrolling. The styleguide does this.

## House rules

These are not preferences. They are what separates this system from generic
modern UI, and anything on this list is a defect:

- `border-radius` above `0`. No pills, ever.
- Gradients of any kind, linear or radial.
- `filter: blur()`, `backdrop-filter`, or any soft/alpha drop shadow. Shadows
  are opaque offset blocks — `4px 4px 0 0 var(--primary)`.
- Eased transitions on interactive state. Printed objects do not ease; buttons
  press *into* their own shadow via `translate(4px, 4px)`.
- Outlined display type (`-webkit-text-stroke`). Display type is solid heavy
  weight — National Park 800, uppercase, `0.05em` tracking.
- `opacity` as a way to express a muted state. Use `--text-muted`.

## Contrast

Both golds are too light to carry text on paper — tighter than the rust they replaced,
so the rule matters more, not less. This constraint is load-bearing: it is why the
points tie keeps its own dark green (`--highlight`) rather than becoming a gold stamp.

| Pair | Ratio | Rule |
|---|---|---|
| `--text-primary` on `--background` | 13.2:1 | Default body text |
| `--text-primary` on `--accent` | 9.3:1 | Ink on sun gold |
| `--text-primary` on `--surface` | 8.5:1 | Text on the slate zebra rows |
| `--text-primary` on `--secondary` | 6.6:1 | Ink on antique gold |
| `--on-highlight` on `--highlight` | 5.4:1 | The points-tie block |
| `--on-primary` on `--primary` | 4.8:1 | Pale yellow on blue — bold headers only |
| `--text-muted` on `--background` | 4.6:1 | Muted cells only — just passes AA |
| `--secondary` on `--background` | **2.0:1** | Rules, fills, borders only. **Never text.** |
| `--accent` on `--background` | **1.4:1** | Fill only. **Never text.** |

## Components

`.band` · `.unigrid` / `.unigrid--grid` / `.unigrid__cell` · `.btn` (+ `--primary`,
`--secondary`, `--ghost`) · `.badge` (+ `--paper`, `--green`, `--rust`) ·
`.rule--accent` / `.rule--heavy` · `.status` + `.dot` (`.live` / `.error`) ·
`.plaque`. See the styleguide for each.

## Notes specific to the leaderboard

- **The background is switchable.** `<html data-scene="…">` in `../index.html` takes
  four values:
  | Value | What it is |
  |---|---|
  | `shore` | Default. Flat sun, sand, dune grass and edge conifers, full bleed. |
  | `ridge` | Flat mountain ridge band pinned to the bottom edge. Token-driven, so it recolors with the palette. |
  | `classic` | The original pre-theme illustration, preserved verbatim as a rollback path. Keeps its own hardcoded colors, gradients and grain filter by design — **exclude it from any theme audit.** |
  | `none` | No illustration, flat sky. |
- **The masthead is deliberately NOT a `.band`.** It floats on the sky so the sun and
  treetops read behind it — that is the only reason the illustration is visible at all
  on a locked one-screen layout where the board covers the middle. Putting a fill back
  on `.masthead` buries the artwork. The `.band` archetype lives on `thead tr.cols`.
- **The shore scene is composed for two strips**, not for the whole viewport: the
  ~150px above the board (sky, sun, treetops) and the ~60px below it (sand, grass),
  plus the narrow side margins (tree trunks). Its viewBox aspect is 2:1, close to a
  1512×743 screen, so `slice` crops only a few pixels. On phones the mobile block
  re-pins it to the bottom at its native aspect, because a portrait viewport would
  otherwise slice it down to the middle ~23% and throw away the sun and both treelines.
- **Column widths live in `<col>` rules**, not on cells. `table-layout: fixed`
  takes widths from the first row, and the first row is the `colspan`'d group
  band, so per-cell widths are ignored and every column comes out equal.
- **The zebra rule is `td:not(.tie)`** so the tie highlight wins without
  `!important`. Do not "simplify" it back to a plain `td`.
- The table is string-built in JS and re-rendered every 30s; class names there
  and selectors here must stay in sync.
- **The board can clip rows.** `.board` is `flex: 0 1 auto` inside a `100vh` column
  whose overflow is hidden, so if the roster outgrows the space the last row is cut
  rather than the footnote being pushed off-screen. Eight teams currently fit with
  ~4px to spare at 1512×743. Adding teams means reclaiming vertical budget — the
  levers, in order of least visual cost: `tbody td` padding, `--fs-display`,
  the two `thead` paddings.

---

## Pushing this to Claude Design

Not done yet, and not a small step. `/design-sync` pushes a **React** design
system: its converter bundles compiled component code from a Storybook setup or
a bare npm package into `_ds_bundle.js` and uploads that. It cannot consume a
CSS file or a static HTML page.

Two blockers as of 2026-08-21:

1. **No Node toolchain on this machine.** No `node`, `npm`, `npx`, `pnpm`,
   `yarn` or `bun` on `PATH`, and no nvm or Homebrew install. The converter runs
   as `node package-build.mjs`.
2. **No React package exists.** This repo has zero dependencies and no build
   step, deliberately.

The follow-on shape, when it's wanted:

1. `brew install node`
2. Scaffold a React + TypeScript package exporting `Band`, `UnigridCard`,
   `Button`, `Badge`, `Status` and `DataGrid` over these same tokens — import
   `wpa.css` rather than re-declaring any value, so this file stays the source.
3. Build to `dist/`
4. Run `/design-sync` and pin the resulting `projectId` in `.design-sync/config.json`

Note that Claude Design builds its Design System pane card index from a
first-line `<!-- @dsCard group="…" -->` comment in each preview HTML, so the
styleguide sections here map onto cards fairly directly when that time comes.
