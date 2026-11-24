# Tasks

## 1. Concurrent provider search

- [x] 1.1 Refactor `search()` so each attempt builds its provider jobs and runs them through a `ThreadPoolExecutor`. Merge the results with `map`, which keeps submission order, and catch errors per job. Verify with an assert self-check using three fake providers that each sleep 0.5 s, the last of which raises: the attempt finishes in under 1 s, and the results keep provider order even when a later provider answers first.
- [x] 1.2 Measure the live speed-up and verify that `python bin/rom --list -p switch "mario kart 8 deluxe" </dev/null` and `python bin/rom --list "celeste" </dev/null` return the same top results as before, with lower wall time than the baseline (4.4 s and 6.7 s).

## 2. Stale-while-revalidate Switch index

- [x] 2.1 In `ArchiveOrg.switch_index`, serve an expired cache that has items, and spawn a detached `switch_index(force=True)` child guarded by a `.refreshing` marker (15 min). A missing cache, a `ver` mismatch, or `force` still builds inline. Verify by backdating `fetched_at` by 8 days: a Switch `--list` search returns within a few seconds, the marker exists, a second search within 15 min spawns no new child, and after the child finishes `fetched_at` is recent.
- [x] 2.2 Update `README.md` (Switch index caching note) and `AGENTS.md` (`search()` concurrency, index refresh and marker in the caches table). Verify that both describe the background refresh and that the README no longer says the first search after expiry takes minutes.

## Workflow follow-up

- Archive the change with `/opsx-archive` after review.
