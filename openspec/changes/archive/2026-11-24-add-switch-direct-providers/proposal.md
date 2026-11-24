# Proposal

## Why

Switch search now finds titles, but most results cannot be downloaded by `rom`.
dlarchive closed its API, romsfun has no Switch files, and nswgame posts mostly link
hosts behind Cloudflare or captchas. Live probes (2026-11-24) found four paths that
serve real bytes to a plain stdlib HTTP client, so `rom -p switch` can download again.

## What Changes

- Add **romslab** (`romslab.com`) as a Switch provider. It has 9.4k posts, and its
  "Mirror 2" filekeeper links resolve to a signed CDN URL in one GET.
- Add an **Archive.org file-level Switch index**. It lists the `.nsp`/`.nsz`/`.xci`
  files inside large public multi-game items, so a search matches filenames and not
  only item titles. It skips login-restricted items.
- Teach the host-link resolver (dlpsgame and nswgame) to download from **pixeldrain**
  (`/api/file/<id>?download`) and to try **mediafire** alongside the existing hosts.
- Keep RAR archives as downloaded. `rom` neither extracts nor converts them. A `.rar`
  is a valid download when it starts with the `Rar!` signature.
- When no host can be fetched directly, keep the current behavior: show the manual
  links.

## Capabilities

### New Capabilities
- `switch-downloads`: how `rom` finds Switch titles and downloads them directly. This
  covers provider coverage, which hosts are fetched, how files are picked, archive
  handling, and the fallback when a host needs a browser.

### Modified Capabilities

## Impact

- `bin/rom`:
  - New `RomsLab` source.
  - New Archive.org Switch index with its cache file.
  - Pixeldrain handler in `DlpsGame.resolve`.
  - `rar` signature check in `validate`.
  - `PLATFORMS["switch"]` provider list.
- New cache file `~/.cache/rom-cli/switchindex.json`. `--index` and `--refresh`
  rebuild it.
- `README.md` and `AGENTS.md`: list the Switch providers and their limits.
- No new dependencies; the code stays stdlib-only.
