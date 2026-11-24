# Design

## Context

See proposal.md for the failing case and the measurements.

`switch_index()` currently works like this:
1. Runs `index_query` (software only, at least 3 files, at least 500 MB) and gets 357 identifiers.
2. Fetches `/metadata/<id>` for each one on 6 threads.
3. Keeps loose `.nsp/.nsz/.xci` entries as `{id, name, size}` in `switchindex.json` (`ver` 1).

`index_results()` matches query tokens against the pretty title derived from the filename, groups the matches by base Title ID (`_group_result`), and builds download URLs as `/download/<id>/<name>`.

Archive.org serves zip members at `/download/<id>/<zip>/<member>`. The member list is the HTML table on `/download/<id>/<zip>/`. Each row has `href="//archive.org/download/<id>/<zip>/<member>"`, and its size is in the trailing `<td id="size">N</tr>`.

## Goals / Non-Goals

**Goals:**
- Find the packages of a Title-ID-named zip in a data item, and download single members from it.
- Keep `-y`/defaults behavior and the Title-ID grouping working on zip members.

**Non-Goals:**
- Opening `.7z` or `.rar` archives. Archive.org lists zip contents only.
- A Title-ID-to-name database. Names come only from data already indexed or already returned by providers.

## Decisions

1. **Wider discovery query.**
   ```
   (mediatype:software OR mediatype:data) AND -collection:loggedin AND
   (subject:(nsp OR nsz OR xci OR switch) OR title:(nsp OR nsz OR xci OR switch) OR identifier:0100*)
   AND files_count:[1 TO *] AND item_size:[300000000 TO *]
   ```
   This gives 732 items. `rows=1000`, sorted by `item_size desc`.

2. **Which zips to open.**
   - A `.zip` is opened when its name has a Title ID, or the item identifier starts with `0100-`, or the item has no loose package and at most 30 zips.
   - Measured during apply: without the 30-zip cap, the rule planned 6,298 listing calls. Three `*-nsw2u-pack` items of other consoles hold 1,520 to 1,957 zips each, and the first rebuild timed out after 20 minutes.
   - The listing is fetched inside the same per-item worker, so the 6-thread pool still bounds concurrency.
   - Members matching `\.(nsp|nsz|xci)$` become entries `{id, name: <member basename>, size, zip: <zip name>, path: <member path>}`.
   - Alternative considered: reading the zip central directory with HTTP Range requests. Rejected, because the listing page already gives names and sizes in one request.

3. **Titles for Title-ID-only names.**
   - Each entry stores `title`:
     - the pretty title from the filename, when it contains letters other than the Title ID;
     - otherwise the item title, when that is not just a Title ID;
     - otherwise empty.
   - After the crawl, entries with an empty title copy the title of any other entry with the same base Title ID. When no title can be found, the entry stays searchable by Title ID only.
   - `index_results(query, tids=())` matches an entry when every query token is in its title, or when its base Title ID is in `tids`.

4. **Title IDs from providers, fetched lazily.**
   - Trigger: in `search()` for `switch`, the token match returns no Title-ID group whose title contains every query token.
   - First, Title IDs are read from that search's results (titles, pages, filenames).
   - If there are none, the top-ranked nswgame post's links are fetched once and their `[0100…]` IDs are extracted. nswgame links carry Title IDs (for example `..._[0100AE00096EA000]+[v1.0.1]...`), while post URLs rarely do.
   - The index is then re-queried with `tids`.
   - Cost: at most one extra request, and only when the token match misses.
   - Alternative considered: always fetching nswgame links during search. Rejected, because it adds one request to every Switch search.

5. **Download URL for members.** `/download/<id>/<quote(zip)>/<quote(path)>`. `ArchiveOrg.resolve` already returns `meta["url"]` for grouped choices, so this only changes how the URL is built.

6. **Non-Switch filter for item search.** `ArchiveOrg.search(query, "switch")` drops docs whose title matches `(?i)\b(pc|recomp\w*|port|windows|android)\b`, unless the title also contains `nsp|xci|nsz`. No metadata fetch is needed.

7. **Cache shape.** Entries gain `title`, `zip` and `path`; `INDEX_VER` goes to 2. A version mismatch triggers an inline rebuild (existing behavior), which happens once.

## Risks / Trade-offs

- [The first search after the upgrade rebuilds inline: about 730 metadata calls (about 140 s on 6 threads) plus the capped zip listings] → a one-time cost, with a progress line shown; the README notes it.
- [The 30-zip cap could skip a large Switch pack whose zips lack Title IDs] → such packs usually name their zips with TIDs, which bypasses the cap. The cap is a `ponytail:` constant to raise if a pack is missed.
- [Wider query pulls in non-Switch items] → only `.nsp/.nsz/.xci` files and members are indexed, so noise items add no entries.
- [The PC/port word filter could hide a real Switch item titled "… Port"] → `switchports` uses a separate path (`port_results`), and items with package extensions in their titles are kept.
- [Zip listing HTML changes] → if parsing fails, that zip contributes nothing and the rest of the index is unaffected.
