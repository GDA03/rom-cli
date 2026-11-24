# Proposal

## Why

A headless TUI run on 2026-11-24, with screenshots of the picker, a "mario kart" search, marked rows, the palette and a platform switch, showed that the result table carries little useful signal and some keys behave unexpectedly:

- ★ is on 22 of the 26 visible rows, hacks included.
- Size and Type are almost always `?`/`-`.
- Marked rows are dark text on green and hard to read.
- Enter leaves a stray mark behind.
- Switching platform after a search re-runs the old query on the new platform.

These are small, contained fixes that make the existing TUI usable before larger features land.

## What Changes

- ★ marks a result only when its title is an exact (normalized) match for a Wikipedia best-seller and is not a hack. This also changes which results get the best-seller ranking boost in both the CLI and the TUI.
- Each row shows whether its source downloads directly or only yields links for a browser.
- Size and Type columns are hidden when no visible row has a value. The title column takes the freed width.
- Marked rows stay readable: light text on the mark color, with a visible ✔.
- The Portuguese "todas" becomes "All" in the picker and the status bar.
- Enter downloads the highlighted row without marking it. The options palette stays on ctrl+b.
- `p` with an empty search field browses the chosen platform's catalog. With text in the field it keeps searching on the new platform. `/` focuses the search field.
- Key hints move into the Textual footer. The status line only shows messages.
- When search results are cut to 200 rows, the status line says so ("200 of N").

## Capabilities

### New Capabilities
- `best-seller-marking`: which results count as best-sellers. This decides the ★ marker and the ranking boost in both the CLI and the TUI.
- `tui`: the interactive result table. Covers columns, markers, marked-row style, key bindings, the footer and the status line.

### Modified Capabilities

## Impact

- `bin/rom`:
  - `pop_rank()` and `annotate_popular()`
  - `run_tui()`: `PlataformasScreen`, `RomApp` bindings, `_cells`, `_hint`, `_show`, `_render_page`, `_open_platforms`, `on_data_table_row_selected`
  - a `direct` attribute on each source class
- No new dependencies.
- Verified with the headless screenshot harness (`rom-cli-tui-screenshot`) before and after.
