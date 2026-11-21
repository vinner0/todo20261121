# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file to-do tracker: `index.html` (HTML, CSS and vanilla JS inline). No frameworks, no build step, no tests, no package manager, not a git repo. Open the file directly in a browser to run it.

Each item has: item text, priority (High/Medium/Low), deadline (`YYYY-MM-DD`, optional), requested by (optional), and `done`.

`mockup.html` is a generated Galaxy S25 mockup (two phone frames, light and dark, each embedding a seeded copy of `index.html` as an iframe `srcdoc`). It is a static snapshot: never edit it by hand, and regenerate it after changing `index.html`. The generator (`make-mockup.ps1`) lives in the session scratchpad, not in this folder, so it may need to be rewritten. It replaces the `let items = JSON.parse(store('items') || '[]');` line with seed data and the `setTheme(store('theme') ||` line with a forced theme, so keep both lines intact or update the anchors.

## Architecture

- **State**: one `items` array, persisted as JSON in `localStorage` key `items` (theme in key `theme`). All storage goes through the `store(key, val)` try/catch helper, so the app still works when storage is blocked (private browsing, `data:` previews). Never call `localStorage` directly.
- **Rendering**: `render()` rebuilds the whole table body from `items` on every change. It sorts `items` in place (done last, then deadline with empty = last, then priority), builds the summary chips, then filters for display. Mutations follow the pattern `save(); render();`.
- **Filtering** is view state, not data: `view` ('all' | 'open' | 'overdue' | 'today'), the `#search` text (matches item and requestedBy) and `#prioFilter`.
- **Edit/delete use object references** (`editItem`, `items.indexOf(it)`), never row indexes, because the displayed list is filtered and sorted differently from the array.
- **Dates**: use `todayStr()` (local date). Never `toISOString().slice(0, 10)`, which is UTC and is a day off in Singapore before 8am.
- **Theme**: CSS variables on `:root`, overridden by `:root[data-theme=dark]`. `setTheme()` sets the attribute and stores the choice; the default follows `prefers-color-scheme`.
- **Mobile**: under `@media (max-width: 640px)` the table rows become grid cards. Inputs are 16px (avoids iOS zoom on focus) and tap targets are 44px. The `Due:` and `From:` labels come from `td:nth-child(4)` and `td:nth-child(5)` `::before`, so changing the column order breaks them.
- **Import/Export**: export downloads a JSON array. Import validates the shape, defaults a bad priority to Medium, drops a malformed deadline, and skips duplicates (same item + priority + deadline + requestedBy).

## Gotchas

- Items created through the form have no `done` field (undefined is treated as false).
- On mobile the add form is a 2-column grid (`#item` spans both; priority + date on row 2; requester + Add on row 3). The `form > * { width:100%; min-width:0; }` rule keeps the grid cells from overflowing. Don't go back to a flex column: `#item`'s flex-basis makes the input tall there.
- The Browser pane renders local files as static snapshots and is light-scheme only. To check dark mode or a phone width there, call `setTheme('dark')` and resize the viewport. Export/Import (file dialogs) can't be tested there.
