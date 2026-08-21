# WPA-National-Park-Vintage

A design system for printed-badge UI: 1930s–1970s National Park Service "Unigrid"
layout rules, WPA poster color, flat offset block shadows. Built for the Jambo
Leaderboard and intended to move to Claude Design.

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
<link href="https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="design-system/wpa.css">
```

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

Two palette colors cannot carry text on cream. This constraint is load-bearing:
it is why the points-tie highlight is charcoal on mustard rather than the reverse.

| Pair | Ratio | Rule |
|---|---|---|
| `--text-primary` on `--background` | 12.6:1 | Default body text |
| `--text-primary` on `--surface` | 11.1:1 | Text on tan cards and zebra rows |
| `--background` on `--primary` | 8.1:1 | Cream on the green band |
| `--text-primary` on `--accent` | 7.8:1 | The highlight pattern |
| `--text-muted` on `--background` | 4.7:1 | Muted cells only — just passes AA |
| `--text-primary` on `--secondary` | 4.4:1 | Charcoal on rust |
| `--secondary` on `--background` | **2.9:1** | Rules, fills, borders only. **Never text.** |
| `--accent` on `--background` | **1.6:1** | Fill only. **Never text.** |

## Components

`.band` · `.unigrid` / `.unigrid--grid` / `.unigrid__cell` · `.btn` (+ `--primary`,
`--secondary`, `--ghost`) · `.badge` (+ `--paper`, `--green`, `--rust`) ·
`.rule--accent` / `.rule--heavy` · `.status` + `.dot` (`.live` / `.error`) ·
`.plaque`. See the styleguide for each.

## Notes specific to the leaderboard

- **The background is switchable.** `<html data-scene="…">` in `../index.html`
  takes `wpa`, `classic` or `none`. `classic` is the original pre-theme
  illustration, preserved verbatim as a rollback path — it keeps its own
  hardcoded colors and gradients by design, so exclude it from any theme audit.
- **The WPA scene is a bottom band, not a full-bleed image.** On the locked
  desktop layout the board covers nearly the whole viewport, so a full-bleed
  illustration only leaks a few pixels of unreadable color at the margins.
  Sizing it as a band (`.scene[data-name="wpa"] svg`) makes it read in the side
  margins and in the strip beneath the board. Its fills are tokens, so it
  recolors with the system.
- **Column widths live in `<col>` rules**, not on cells. `table-layout: fixed`
  takes widths from the first row, and the first row is the `colspan`'d group
  band, so per-cell widths are ignored and every column comes out equal.
- **The zebra rule is `td:not(.tie)`** so the tie highlight wins without
  `!important`. Do not "simplify" it back to a plain `td`.
- The table is string-built in JS and re-rendered every 30s; class names there
  and selectors here must stay in sync.

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
