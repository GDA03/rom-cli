# Proposal

## Why

"Hyrule Warriors: Definitive Edition" cannot be downloaded although Archive.org has it.

Probes on 2026-11-24:
- romslab offers only datanodes (captcha); nswgame offers megaup (Cloudflare) or dead 1fichier links; multiup lists only blocked mirrors.
- The game is in Archive.org item `0100-ae-00096-ea-000`: one `[0100AE00096EA000].zip` (13.9 GB) holding the `.xci` base and `(Update 1.0.1).nsp`. The zip and each member are served directly (`206 PK`; members `200`, the update starts with `PFS0`).

The Switch index misses this item for three reasons:
- the item is `mediatype:data`;
- its title is only the Title ID;
- the packages sit inside a `.zip`.

The only Archive.org hit shown, "…Recompilación para PC", is a PC port presented as a Switch result. Widening the query to `mediatype:data` and Title-ID identifiers grows it from 357 to 732 items. In a 60-item sample there were 39 loose packages, 163 archives, and 17 archives named with a Title ID.

## What Changes

- The Switch index also covers `mediatype:data` items, items whose identifier is a dashed Title ID (`0100-…`), and smaller items (at least 300 MB, at least 1 file).
- Archives (`.zip`) in indexed items are opened through Archive.org's archive listing. Their `.nsp`/`.nsz`/`.xci` members become indexed files that download directly from inside the zip.
- Indexed files get a title even when their name is only a Title ID: the item title, or a title learned from another file with the same Title ID. A search also matches by Title ID when a provider result names one, so "hyrule warriors definitive edition" finds the item.
- Archive.org item-search results that are not Switch packages, such as PC ports, recompilations, or items without any Switch package or archive, are no longer shown as Switch results.

## Capabilities

### New Capabilities

### Modified Capabilities
- `switch-downloads`: the "Archive.org Switch file index" requirement now covers data items, Title-ID items and packages inside zip archives. A new requirement excludes non-Switch Archive.org hits.

## Impact

- `bin/rom`:
  - `ArchiveOrg.index_query`;
  - `switch_index()`, which reads zip listings and stores a `zip` path for members;
  - `index_results()`, which matches by title or Title ID;
  - `ArchiveOrg.search()`, which filters out non-Switch hits;
  - Title-ID lookups from nswgame and romslab results.
- `switchindex.json` changes shape (new `zip` field, a title per entry). Bump `INDEX_VER`; the first search after upgrading rebuilds the index inline.
- The index build takes longer: about 730 metadata calls plus one listing call per zip, still on 6 threads, at most once a week in the background.
