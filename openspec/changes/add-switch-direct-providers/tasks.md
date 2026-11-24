# Tasks

## 1. Archive handling

- [x] 1.1 Add a `.rar` branch to `validate()` that requires the `Rar!\x1a\x07` signature (keep the >50 MiB shortcut). Verify with an assert self-check that a small file starting with `Rar!\x1a\x07` passes and an HTML stub named `.rar` fails.
- [x] 1.2 Change the `nswgame` penalty in `rank()` from −10 to −12. Verify with an assert that a romslab result outranks an nswgame result with the same title.

## 2. romslab source

- [x] 2.1 Add a `RomsLab` source class: WP REST search filtered to Switch slugs (exclude `switch-2`), `Result` with platform `Switch`, `pk` `switch`, `ext` `rar`. Register it in `SOURCES`. Verify `python bin/rom --list -p switch "mario kart 8 deluxe" </dev/null` shows a romslab hit once 2.3 is wired.
- [x] 2.2 Implement `RomsLab.resolve`: scrape the post's `Mirror 2` filekeeper link and return the `op=download2…&down_direct=1` URL. With no mirror, raise `RuntimeError` listing the post's `btn-download` links. Verify `python bin/rom --print-url -n 1 -p switch "hollow knight" </dev/null` prints a filekeeper URL, and that a `Range: bytes=0-15` GET on it returns 206 `Rar!`.
- [x] 2.3 Put `("romslab", "switch")` first in `PLATFORMS["switch"]`, and add romslab to the first attempt of the no-platform `search()` branch. Verify `python bin/rom --list -n 10 "celeste" </dev/null` includes Switch results.

## 3. Archive.org Switch file index

- [x] 3.1 Add the item discovery (advancedsearch query from design decision 2) and the metadata crawl (6 threads, restricted and private items skipped). Write `switchindex.json` with `fetched_at`, `ver` and `items`, using the tmp + `os.replace` pattern and a 7-day TTL. Verify `python bin/rom --index` prints a non-zero Switch file count and the cache file exists.
- [x] 3.2 Add a file-level search over the index: normalized filename token match, Title ID based Base/Update/DLC, and `Result.page = /download/<id>/<quoted name>` so `resolve` returns it unchanged. Verify `python bin/rom --list -p switch "hollow knight" </dev/null` shows an archive.org `.nsp` with type `Base`.
- [x] 3.3 Wire the index as an `archive.org` provider for `switch`, ahead of item search. Make `--index`/`--refresh` rebuild it. Verify `python bin/rom --print-url -n 1 -p switch "celeste" </dev/null`, then a `Range` GET on the printed URL returns 206 `PFS0`.

## 4. Host resolver

- [x] 4.1 Add a `_pixeldrain` handler and the `pixeldrain.com` host. Reorder the `DlpsGame.resolve` host priority to pixeldrain, mediafire, google, 1fichier, then the rest. Verify with a direct call on a known nswgf post that links pixeldrain: `resolve` returns `https://pixeldrain.com/api/file/<id>?download`.
- [x] 4.2 Do one real end-to-end download of a small Switch file into a temp dir (pixeldrain or a small archive.org `.nsp`). Verify it ends in `✔ done` and passes `validate()`. Delete the temp dir afterwards.

## 5. Docs

- [x] 5.1 Update `README.md` ("How it works", Switch bullet, caches) and `AGENTS.md` (sources list, caches table, host-link limits) to describe romslab, the Archive.org index, pixeldrain and RAR handling. Verify that no remaining text claims RAR files get extracted, and that `switchindex.json` is listed.

## Workflow follow-up

- Archive the change with `/opsx-archive` after review.
