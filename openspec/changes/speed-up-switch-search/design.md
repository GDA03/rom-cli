# Design

## Context

See proposal.md for the measurements.

`search()` loops over query variants ("attempts"). It stops at 12 results, and each attempt calls its providers in sequence. Within one Switch attempt that means romslab (about 1.2 s), the archive.org index (about 0.03 s warm), archive.org item search (about 1.3 s), then nswgame (about 1.9 s).

`ArchiveOrg.switch_index()` loads `switchindex.json` when it is under 7 days old and otherwise rebuilds it inline: one advancedsearch query plus about 357 `/metadata` calls on 6 threads, about 94 s in total.

`rom` is a short-lived CLI, and the TUI is a long-lived process.

## Goals / Non-Goals

**Goals:**
- Make an attempt take as long as its slowest provider.
- Make no search block on an index rebuild, except the very first build.

**Non-Goals:**
- Running attempts concurrently. The early stop at 12 results depends on order.
- Speeding up `stream()`.
- Changing the provider list, ranking or TTLs.

## Decisions

1. **One thread pool per attempt; results merged in provider order.**
   - Each attempt builds a list of `(callable)` jobs:
     - with a platform: one per `PLATFORMS[key]` entry;
     - without a platform: romsgames and romsfun, plus NPS, dlpsgame, nswgame and romslab on the first attempt.
   - The jobs run with `ThreadPoolExecutor(max_workers=len(jobs)).map`. `map` yields results in submission order, so `add()` sees the same sequence as the old serial loop.
   - Each job catches its own exception and returns `[]`, keeping the current skip-on-failure behavior.
   - Alternative considered: one pool for the whole search across attempts. Rejected, because the early stop at 12 results would change which attempts run.
   - Thread safety:
     - `NoPayStation.load` already holds a lock.
     - Each attempt calls the switch index once, so it has no concurrent callers inside a search.
     - `on_progress` → `status()` may interleave status lines, which is cosmetic.

2. **Stale-while-revalidate for the Switch index, refreshed by a detached child process.**
   - In `switch_index(force=False)`, a cache file older than `INDEX_TTL` but holding items is served. A refresh is then started unless the marker `switchindex.json.refreshing` has an mtime under 15 minutes old.
   - The marker is written (touched) before spawning the child.
   - The child is `sys.executable -c "<load bin/rom via SourceFileLoader>; SOURCES['archive.org'].switch_index(force=True)"`, with stdio sent to `DEVNULL`. On Windows it is detached with `DETACHED_PROCESS | CREATE_NEW_PROCESS_GROUP`; on POSIX with `start_new_session=True`.
   - The existing tmp + `os.replace` write keeps readers from ever seeing a partial file.
   - The marker is never deleted. A fresh `fetched_at` stops further spawns, and the 15-minute window limits retries after a crashed child.
   - Alternative considered: a daemon thread. Rejected, because a 5 s CLI run exits long before a 94 s rebuild finishes, so the index would never refresh outside the TUI.
   - Alternative considered: a new hidden CLI flag. Rejected, because the `-c` one-liner reuses `switch_index(force=True)` without widening the CLI.
   - A missing cache file, a different `ver`, `--refresh` and `--index --refresh` still build inline (`force=True`).

## Risks / Trade-offs

- [The child process is killed or offline] → the stale index keeps working, and the next search after 15 minutes retries.
- [The stale index lists files that were taken down] → `stream()` fails with an HTTP error on that one result. This already happened under the 7-day TTL.
- [Thread pools add CPU and socket overhead] → a pool has at most 6 workers and lives for one attempt, which is negligible next to HTTP latency.
- [Status lines interleave while NPS loads concurrently] → this is cosmetic, and NPS loads only once per 7 days.
