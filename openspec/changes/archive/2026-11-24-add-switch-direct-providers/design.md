# Design

## Context

`bin/rom` is one stdlib-only Python script.

- Providers are duck-typed sources registered in `SOURCES`. `PLATFORMS[key]` holds an
  ordered `[(source, arg)]` list.
- `search()` merges the results of every provider. `rank()` orders them.
- `download()` calls `resolve()` and then `stream()`, which writes a `.part` file and
  resumes with `Range`. `fix_magic_ext()` and `validate()` run after the stream.
- Today `switch` maps to `archive.org` item search and `nswgame`.

Every path below was probed on 2026-11-24 with Python urllib, using a 16-byte `Range` GET:

| Path | Request chain | Result |
|---|---|---|
| romslab → filekeeper | WP search, then the post HTML `Mirror 2` link, then `GET https://filekeeper.net/download?op=download2&id=<code>&method_free=Free+download&down_direct=1` | 302 to `edge01.fetchio.cc/f/…?sig=…`, 206 `Rar!` (MK8DX 7.49 GB) |
| Archive.org file | `/download/<id>/<file>` | 302 to a storage node, 206 `PFS0` (Celeste 1.18 GB) |
| pixeldrain | `/api/file/<id>?download` | 206 `Rar!` |
| mediafire | page scrape for the `download\d+.mediafire.com` href | 206 `Rar!`; some pages return 403 |

Hosts we ruled out:
- Cloudflare blocks MegaUp, send.now, qiwi, buzzheavier and ouo.io.
- datanodes, vikingfile and mixdrop use Turnstile; clicknupload uses hCaptcha.
- gofile needs an obfuscated token.
- 1fichier needs a 60-second wait plus a POST, and the links we tried were dead.
- mega.nz encrypts files with AES, which the stdlib cannot decrypt.

## Goals / Non-Goals

**Goals:**
- Resolve Switch downloads with stdlib HTTP only. Each resolve takes at most three requests and does not wait on a timer.
- Rebuild the Archive.org index from search queries instead of a hardcoded list of identifiers, because items get taken down.

**Non-Goals:**
- Solving captchas, emulating JS or TLS fingerprints, or using accounts and premium tiers.
- Extracting archives, or converting NSZ to NSP.
- Downloading romslab updates and DLC. These are on datanodes only, which is behind a captcha.
- Downloading multi-part RAR sets automatically. Each part is a separate result or link.

## Decisions

1. **romslab is its own source class, not a `DlpsGame` subclass.**
   - romslab puts the download buttons in its theme HTML, and they are missing from `content.rendered`. It also has one host flow, filekeeper.
   - Search uses `/wp-json/wp/v2/search?search=<q>&per_page=20`. Keep only URLs whose slug contains `switch`, and drop `switch-2` (Switch 2) posts.
   - `resolve` fetches the post and matches `btn-download" target="_blank" href="(https://filekeeper\.net/([a-z0-9]{12})/[^"]+)">Mirror 2<`. It then builds the `op=download2` URL and returns it. `stream()` follows the 302 itself, so the signed URL is fetched right away, before it expires.
   - If no filekeeper mirror exists, raise `RuntimeError` listing the post's `btn-download` hrefs.
   - Alternative considered: walk filekeeper's two-step `download1`/`download2` form flow. Rejected, because the single `down_direct=1` GET already returns the redirect.

2. **Archive.org Switch index: discover the items, then build a file-level cache.**
   - Find the items with advancedsearch:
     `mediatype:software AND -collection:loggedin AND (subject:(nsp OR nsz OR xci OR switch) OR title:(nsp OR nsz OR xci OR switch)) AND files_count:[15 TO *] AND item_size:[5000000000 TO *]`
     sorted by `item_size desc`, limited to 200 rows.
   - For each item, fetch `/metadata/<id>` with 6 threads. Skip the item when `access-restricted-item` is set or any file is `private`.
   - Keep files matching `(?i)\.(nsp|nsz|xci)$`. Store `{id, name, size}` plus derived fields in `switchindex.json`, with a 7-day TTL and a `ver` key. This follows the existing `switchports.json` pattern.
   - Search matches tokens against the normalized filename. Normalizing drops `[...]`, `(...)`, `_` and the extension.
   - Item-title search (`ArchiveOrg.search` with the `switch` tag) stays as a second tier for single-game items.
   - Alternative considered: a hardcoded list of known-good identifiers. Rejected, because uploads move into `loggedin` without notice.
   - Alternative considered: advancedsearch full text. Rejected, because it does not index filenames.

3. **Base, Update and DLC are derived from the Title ID.**
   - When the filename contains `[0100XXXXXXXXX000]`, a suffix of `000` means Base, `800` means Update, and anything else means DLC.
   - Otherwise fall back to the current `kind_of()`, which reads text tags.
   - `KIND_ORDER` and `rank()` already prefer Base.

4. **Host resolver gains pixeldrain, and mediafire moves first.**
   - Add `_pixeldrain` to `DlpsGame`: regex `pixeldrain\.com/u/(\w+)`, then return `https://pixeldrain.com/api/file/<id>?download`.
   - Add `pixeldrain.com` to `HOSTS`.
   - Resolve order becomes pixeldrain, mediafire, Google Drive, 1fichier, then the hosts we can only list.
   - `NswGame` inherits all of this.

5. **RAR is validated, not extracted.**
   - `validate()` adds a `.rar` branch that requires `Rar!\x1a\x07`. The existing over-50 MiB shortcut stays.
   - `fix_magic_ext()` already maps `Rar!` to `.rar`.
   - `ext_for()` picks up `.rar` from the URL or filename, and `ROM_EXTS` already lists `rar`.

6. **Ranking.**
   - Leave `romslab` without a penalty, keep archive.org at −8, and change `nswgame` from −10 to −12.
   - With these values a romslab hit outranks the same title on nswgame and on archive.org item search. A file-index hit has a clean filename, so it competes on title match alone.

7. **Provider list.** `PLATFORMS["switch"]` becomes `romslab`, `archive.org` (file index), `archive.org` (item search), `nswgame`. The no-platform branch of `search()` adds romslab on the first attempt, the same way nswgame is added.

## Risks / Trade-offs

- [filekeeper adds Cloudflare or a captcha] → `resolve` raises an error that includes the links. The romslab entry is still useful as a link source.
- [The signed filekeeper URL expires in minutes] → resolve happens immediately before `stream()`. A resumed `.part` re-resolves on the next run, because `download()` always calls `resolve()` first.
- [Archive.org items get restricted after indexing] → `stream()` gets a 401 and the download fails with an HTTP error. `--refresh` rebuilds the index.
- [Building the index is slow: about 200 metadata calls] → it runs only when the cache is missing or older than 7 days, and when `--index`/`--refresh` is used. It shows a progress line like the NPS loader.
- [Large RAR files over 50 MiB skip the signature check] → this is existing behavior. Keep it, because `fix_magic_ext()` still renames the file by its magic bytes.
- [romslab moves to another domain] → `base` is one constant.

## Migration Plan

- No user data migration is needed. The new cache file builds itself on first use.
- Rollback means reverting the commit; the old cache files are not touched.
