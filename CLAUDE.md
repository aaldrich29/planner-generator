# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file HTML productivity planner generator (`index.html`) — no build system, no dependencies, no npm. Open directly in a browser. Changes take effect on reload.

Published via GitHub Pages from the repo root, which is why the file is named `index.html`.

The local backup copy `planner - Copy.html` is gitignored and not kept in sync; edit only `index.html`.

## Architecture

Everything lives inline in `index.html`: CSS in `<style>`, JavaScript in `<script>`. The file is ~2,900 lines with clear `─────` section-separator comments throughout.

**Three UI regions:**
- **Top bar** — parameter inputs (date, week, month, quarter, project, person, topic) and action buttons
- **Left sidebar** (185px) — 6 collapsible section groups with page-type nav buttons
- **Preview area** — WYSIWYG preview of the current page, scaled to fit viewport at ≤100%

**Data flow for rendering a page:**
1. User clicks a sidebar button → `pick(btn)` fires
2. `pick()` shows/hides relevant top-bar controls via `data-ctrl` attributes on each button
3. `renderCurrent()` looks up the page type in `PAGE_MAP` and calls its renderer function
4. The renderer reads DOM input values (`document.getElementById('inp-*').value`) and returns an HTML string
5. The string is injected into `#preview` and `rescale()` zooms to fit

## Key Structures

### PAGE_MAP (line ~2151)
Registry mapping page-type IDs (strings) to renderer functions. Add an entry here when creating a new page type.

### Page renderer functions
Named `pg[PageName]()`, returning template literal HTML. Each function reads inputs directly from the DOM. The output should be a single `<div class="planner-page ...">` element.

### Parameter controls
Top-bar inputs use IDs like `inp-date`, `inp-week`, `inp-month`, `inp-year`, `inp-quarter`, `inp-project`, `inp-person`, `inp-topic`. Each sidebar button declares which controls it needs via `data-ctrl="week date"` (space-separated list). `pick()` hides all control groups then shows only the declared ones.

### Print Queue system (lines ~2310–2827)
- `addToQueue()` — snapshots current page type + all parameter values + margin mode
- `renderQueue()` — redraws the queue sidebar (drag-reorder, margin badges, back-mode toggles)
- `printQueueItems()` — renders all queued pages into a hidden container and calls `window.print()`
- Queue items stored in the `queue` array; each item is a plain object `{ pageId, params, margin, back }`

### Shared CSS utility classes
| Class | Purpose |
|---|---|
| `.wline` | Single write line (bottom border) |
| `.cb-row` | Checkbox row |
| `.num-row` | Numbered line |
| `.pg-title` | Page header/title block |
| `.sec-label` | Section label (small caps) |
| `.pt` | Planner table (full-width, consistent borders) |
| `.box-border` | Bordered box region |

### Margin modes
Each page carries a margin class: `margin-left`, `margin-right`, or `margin-center`. Toggle per queue item via the R/L/C badge. Default margin is controlled by the top-bar segmented control.

### localStorage persistence
`STORAGE` object (prefix `plnr_`) persists: user name, accent color, line style, favorites array, and saved queue presets.

## Binder System Context

The page types map to a 5-section physical binder:

| Section | Key page types |
|---|---|
| 01 Navigation | monthly-calendar, annual-deadlines, brain-dump |
| 02 Execution | weekly-cc, daily-slip, weekly-review, weekly-metrics |
| 03 Projects | project-dashboard, milestone-tracker, next-actions, project-log |
| 04 People | waiting-on, delegation, one-on-one, meeting-notes |
| 05 Reference | sop-ref, wins-tracker, habit-tracker, notes-page, divider-page |

Strategic pages (life-compass, annual-goals, quarterly-plan, etc.) sit outside the 5-section core but are available from the sidebar.

## Adding a New Page Type

1. Write a `pgMyPage()` renderer function that returns a `<div class="planner-page">` HTML string.
2. Add `'my-page': pgMyPage` to `PAGE_MAP`.
3. Add a `<button data-page="my-page" data-ctrl="...">` in the appropriate sidebar section.
4. Add any page-specific CSS in a clearly labeled section block.

## Print Considerations

- Pages are sized at `8.5in × 11in` (letter). Keep all content within printable margins.
- Colors use `#ccc` borders and `#fff` backgrounds — avoid dark fills that waste ink.
- Test print output via browser print preview (Ctrl+P); `@media print` rules hide the app shell.
- Duplex ("Double") mode pairs each queue item with an auto-generated back side; sheet count ≠ page count.
