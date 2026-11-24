# Design

## Context

This change builds on `tui-quick-fixes`, which must be applied first:
- Enter downloads the highlighted row.
- Key hints live in the Footer (`Binding(show=True)`).
- Each source has a `direct` attribute.

Current data model: one `Result` per file or post. `meta` holds `pk`, `ext`, `kind`, `bytes` and `filename`; `title_kind()` already derives Base/Update/DLC from the Title ID suffix. `RomsFun.resolve()` re-reads the download page and picks a file (USA/World non-hack, shortest name). `download()` resolves one result and streams one file. The TUI's `_batch_bg` downloads rows one after another and reports progress in `#status`.

## Goals / Non-Goals

**Goals:**
- One row per game, with versions, updates and DLC chosen in a panel.
- Unchanged non-interactive behavior (`-y`, `d`, Enter) through defaults.

**Non-Goals:**
- Downloading Update/DLC from captcha hosts.
- Mapping `vN` release numbers to marketing versions such as "3.0.3". There is no such database, so we show "rev N" or a version tag already in the name.
- Parallel downloads. The queue is still sequential; the measured bandwidth is shared anyway.
- Grouping Title-ID-less files by name similarity.

## Decisions

1. **A `Result` carries choices instead of one fixed file.**
   - Add `meta["choices"]`: a list of `{label, kind, size, url|page, direct, default}`.
   - Add `meta["picked"]`: the indexes the user chose.
   - Leave `Result.__slots__` unchanged; `meta` is already a free-form dict.
   - `download()` expands one `Result` into one job per picked choice, or the defaults when nothing is picked.
   - Alternative considered: a new `Game` class. Rejected, because every caller (`rank`, `show`, `fmt_row`, `select_menu`, `run_cli_download`) already speaks `Result`.

2. **Title ID grouping happens at query time, not in the cache.**
   - `index_results()` groups the matched files by `base_tid = tid & ~0x1FFF`. This mask clears the update bit (0x800) and the DLC range (0x1000 plus an index).
   - Each group becomes one `Result`: its title is the base file's pretty title (or the first file's when there is no base), and its `choices` are all of the group's files.
   - `switchindex.json` stays unchanged, so `INDEX_VER` is not bumped.
   - Alternative considered: storing groups in the cache. Rejected, because grouping 548 entries costs microseconds and storing it would force a cache migration.

3. **Version labels.**
   - Release number: `[vN]` divided by 65536 ("rev 23").
   - Version tag: `v\d+(\.\d+)+` in the name ("v1.2.1").
   - Region: the first `[US|EU|JP|…]` or `(USA|Europe|…)` tag.
   - Format: the file extension.
   - romsfun labels come from the cleaned table-row text the resolver already parses: name, tag (No-Intro, Hack, ISO) and size.

4. **romsfun files are listed lazily.**
   - Add `RomsFun.files(result)`. It fetches the game page, follows the download page and returns `[(row_url, label, size)]`. The same parsing currently lives in `resolve()`; move it into `files()`.
   - `resolve()` takes `meta["picked"]` when set, and otherwise the existing default rule.
   - The detail panel calls `files()` on open. Search does not call it, to avoid one extra request per row.

5. **Link-only extras.**
   - romslab: the post's `btn-download` text gives the type ("Download Here" means Base, "Update", "DLC's").
   - nswgame/dlpsgame: the type comes from the URL (`upd|update` means Update; `dlc|booster|wave` means DLC; otherwise Base).
   - These choices are created with `direct: False`. The panel shows them as text and cannot tick them.

6. **Merging duplicate titles.**
   - After `rank()`, rows are merged by `(norm(title), platform)` among results that are not Switch groups.
   - The kept row is the highest-ranked one, and `meta["alts"]` lists the other sources.
   - The panel offers the sources as one more single choice.
   - Merging is done in the TUI only; `--list` and `--print-url` keep one row per source, because scripts rely on them.

7. **Widgets.**
   - `DetailScreen(ModalScreen)`: a `Static` header, a `RadioSet` for the base, file or source, a `SelectionList` for Updates and DLC, and a `TextArea(read_only=True)` for the links, which allows selection and copy.
   - `QueuePanel`: a docked, collapsible `VerticalScroll` with one `ProgressBar` and a label per job.
   - Picker: an `Input` above the `OptionList`, filtered with `on_input_changed`. Fixed head order: Switch, PS4, PS2, GBA, NDS, PSP, then alphabetical.
   - Key binding for the panel: `i`.

8. **Cancellation.**
   - `stream()` gains an optional `cancel` `threading.Event`, checked between 1 MiB chunks. When set, it raises `RuntimeError("cancelled")` and leaves `.part` on disk for resume.
   - The queue sets the event for the selected job.

## Risks / Trade-offs

- [The Title ID bit mask may be wrong for some DLC ranges] → the spec scenario covers base/update/DLC suffixes; verify against the 35 complete groups in the live index, and keep files whose computed base is not found as their own row.
- [`files()` adds latency when the panel opens on romsfun results] → one page fetch plus one download-page fetch, about 1 s; show a spinner in the panel.
- [Merging across sources could join different games that share a title on the same platform] → the platform is part of the merge key; acceptable.
- [`TextArea` copy support depends on the terminal] → links are also printed by `--print-url`.

## Open Questions

- Should the merged row in the CLI `--list` output also show the "+2 upd · 1 DLC" summary? It does not change the TUI design or the tasks, and can be decided when the CLI output is reviewed.
