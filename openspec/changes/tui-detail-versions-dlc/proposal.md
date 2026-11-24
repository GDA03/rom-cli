# Proposal

## Why

Users cannot see or choose what they actually download.

- A Switch game's base, updates and DLC appear as scattered rows. Only the Archive.org index has the type, and it gives nothing to tie the rows to one game.
- Several games have more than one version. TotK has `v0 .nsp`, `v0 .xci` and `v1.2.1 .xci`; 19 indexed games have several update releases; Mario Kart Super Circuit on romsfun has 7 files (regions, Virtual Console, proto, hacks). `rom` silently picks one.
- Link-only sources hide their host links inside an error message.
- Downloads show a single overwritten status line, with no queue and no way to cancel.

Data measured on 2026-11-24:
- Archive.org index: 548 files (136 Base, 148 Update, 40 DLC, 224 without a Title ID); 35 games have a base plus extras; 8 games have several base files.
- romslab and nswgame expose Update/DLC only through captcha or Cloudflare hosts.

## What Changes

- A **detail panel** opens for the highlighted result. It shows the source, the file name or names, the direct/link-only status, and every host link as copyable text.
- **Switch titles grouped by Title ID.** Files from the Archive.org index that share a base Title ID collapse into one result row with a summary such as "+2 upd · 1 DLC". The detail panel lists the Base files, the Updates (newest first) and the DLC. Link-only Update and DLC entries from romslab and nswgame are listed with their type, read from the button text or the URL.
- **Version selection.**
  - In the detail panel the user picks one base file or version.
  - For Switch, the user also ticks any updates and DLC; the newest update is pre-ticked.
  - romsfun results list every file on the game's download page (region, revision, Virtual Console, proto, hack) and let the user pick one.
  - The defaults match today's automatic choice: romsfun USA/World non-hack; Switch `.nsp` base plus the newest update. So `d`, Enter and `-y` behave as before without opening the panel.
- **Download queue panel.** Each queued item shows its own progress bar, keeps its error visible when it fails, and can be cancelled.
- **Duplicate titles across sources** collapse into one row. Their sources become choices in the detail panel.
- **Platform picker.**
  - Switch and the most-used platforms are listed first.
  - Typing filters the list.

## Capabilities

### New Capabilities
- `switch-title-grouping`: how Switch package files are grouped into one game by Title ID, and how Base, Update and DLC are derived and ordered.
- `version-selection`: how the versions and files of one result are listed, which one is the default, and how a choice changes what is downloaded, for Switch and for romsfun.

### Modified Capabilities
- `tui`: adds the detail panel, the download queue, merged duplicate rows and the filterable platform picker. This builds on the `tui` capability that `tui-quick-fixes` introduces; apply that change first.

## Impact

- `bin/rom`:
  - `ArchiveOrg.index_results()` returns grouped results.
  - New helpers parse Title IDs and version tags.
  - `RomsFun` gains a file-listing method; `resolve()` honours a chosen file.
  - `download()` accepts the files picked in the panel.
  - `run_tui()` gets a detail screen, a queue panel, row merging and the picker filter.
- No new dependencies; uses Textual's own `ProgressBar`, `RadioSet`, `SelectionList` and `Input` widgets.
- A cache shape change: if grouping is stored in `switchindex.json`, bump `INDEX_VER`.
