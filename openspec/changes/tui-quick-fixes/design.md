# Design

## Context

All of the TUI lives inside `run_tui()` in `bin/rom`. Its parts:
- the `OptionsScreen` and `PlataformasScreen` modals;
- `RomApp`, which holds `results`, `selected`, `browse_items` and `pk`;
- a `DataTable` whose columns are fixed in `on_mount` and whose cells come from `_cells()`;
- a `#status` Static docked at the bottom, on top of the `Footer`.

`pop_rank()` also matches when a list title is a prefix of the result title. Its result is stored in `meta["pop"]`, which is read by `rank()` (a score boost), `fmt_row()` (the CLI ★) and `_cells()` (the TUI ★).

## Goals / Non-Goals

**Goals:**
- Fix the observed defects with small edits inside the existing structure.

**Non-Goals:**
- Detail panel, download queue, picker filtering, and grouping of duplicate titles. These belong to `tui-detail-versions-dlc`.
- Changing the CLI output layout, beyond the ★ rule it shares with the TUI.

## Decisions

1. **Exact best-seller match.**
   - `pop_rank()` returns a rank only for an exact normalized match; the prefix loop is removed.
   - `annotate_popular()` skips titles that contain any word from `BAD`.
   - This fixes the ★ marker and the ranking boost in one place. Ranking keeps its title-match points, so legitimate subtitled titles still rank near the top.
   - Alternative considered: keep prefix matching for ranking and use exact matching only for ★. Rejected: two rules would make "why is this first" harder to explain, for no measured benefit.

2. **`direct` class attribute on each source.**
   - `True`: romsgames, romsfun, archive.org, nps, romslab (filekeeper is the normal path).
   - `False`: dlpsgame, nswgame.
   - The marker goes in the existing first column next to the mark: `↓` for direct, `↗` for link-only. Both are single-width glyphs; emoji such as 🔗 are double-width and break column alignment.
   - Alternative considered: resolve each row to know for sure. Rejected, because it costs one HTTP round trip per row.

3. **Columns rebuilt per render.** `_show` and `_render_page` call `table.clear(columns=True)` and add only the columns that have at least one value among the displayed rows. `_redraw` updates cells using the current `col_keys`.

4. **Mark style.**
   - The mark cell shows a bold `✔` on `$success`.
   - Marked row cells use bold default-colored text on a muted green background (`on #1f3d2b`), not black text on green.

5. **Keys.**
   - Remove the `on_data_table_row_selected` handler that marked a row and opened the palette. Bind `enter` on the table to `action_download`.
   - Add the binding `Binding("/", "focus_search", "Search")`.
   - `_open_platforms` decides from `Input.value.strip()`: empty means `_start_browse`, otherwise `_start_search`.

6. **Footer and status.**
   - Every action gets a `Binding(..., show=True)` with a short label, so the Footer renders the hints.
   - `_hint()` is reduced to the platform name.
   - `#status` is set to `height: 1` and docked above the footer (it is declared before `Footer` in `compose`).

7. **Truncation notice.** `_show` writes `"200 of N results"` when `len(results) > 200`.

## Risks / Trade-offs

- [Exact matching drops ★ from legitimate retail titles whose names differ slightly from Wikipedia, for example "Pokémon FireRed Version" versus "Pokémon FireRed and LeafGreen"] → acceptable, because the marker now has meaning. A later change can add an alias table if users miss specific titles.
- [romslab is marked direct, but some posts have no filekeeper mirror] → the download then fails with the existing "links:" message. The marker means "usually direct".
- [Rebuilding columns on every render resets the column widths] → this only happens when the rows change. Textual sizes the columns to fit their content.
