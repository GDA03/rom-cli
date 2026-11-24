# Tasks

## 1. Index coverage

- [x] 1.1 Widen `index_query` (software or data, `identifier:0100*`, ≥1 file, ≥300 MB, `rows=1000`) and parse zip listings for Title-ID, `0100-` or package-less items. Store `{id, name, size, zip, path, title}` and bump `INDEX_VER` to 2. Verify with `python bin/rom --index --refresh`: the printed file count exceeds 548 and `switchindex.json` contains an entry with `"zip": "[0100AE00096EA000].zip"`.
- [x] 1.2 Fill empty titles from the item title, then from same-base-TID entries, and build member download URLs as `/download/<id>/<zip>/<path>`. Verify with an assert self-check on a fake index (a TID-only member gets the sibling's title; the member URL is quoted correctly). Also send a live Range GET on the HWDE update member URL from the index: expect `200` and `PFS0`.

## 2. Search by Title ID

- [x] 2.1 Add a `tids` parameter to `index_results`. In `search()` for `switch`, when the token match yields no Title-ID group, collect Title IDs from that search's provider results; if none are found, read them from the top nswgame post's links. Then re-query the index. Verify that `python bin/rom --list -p switch "hyrule warriors definitive edition" </dev/null` shows an archive.org Hyrule Warriors row, and that `--print-url` for it prints an `archive.org/download/0100-ae-00096-ea-000/…zip/…` URL.

## 3. Drop non-Switch hits

- [x] 3.1 Filter `ArchiveOrg.search(…, "switch")` docs whose title marks a PC port or recompilation, unless the title names a package extension. Verify that the "Recompilación para PC" item no longer appears in the HWDE search.

## 4. Docs and end-to-end

- [x] 4.1 Update `README.md` (Switch index covers zips and data items; the one-time rebuild after upgrade) and `AGENTS.md` (index entry fields, zip member URLs, Title-ID search, `INDEX_VER` 2). Run the TUI headless on the HWDE search, open details, and verify that the base and the update appear as choices. Then run `download()` on the update choice with `stream` stubbed and verify that the URL is the member URL, not the whole zip.

## Workflow follow-up

- Archive after review. Archive `tui-detail-versions-dlc` first: both changes touch `switch-downloads`/`tui`.
