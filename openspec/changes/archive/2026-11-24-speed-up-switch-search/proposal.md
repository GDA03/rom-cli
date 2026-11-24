# Proposal

## Why

Measurements on 2026-11-24 show that download throughput is bound by the user's own
network: about 19 MB/s on one connection, with parallel streams adding nothing and curl
matching urllib. The slow moments that `rom` itself causes happen before a download
starts:

- `search()` queries providers one after another. A Switch search takes 4.4 s and an
  all-platform search 6.7 s.
- An expired Archive.org Switch index is rebuilt inline, which blocks the search for
  about 94 s once a week.

## What Changes

- Within each query attempt, providers are queried concurrently, so an attempt takes as
  long as its slowest provider rather than the sum of all providers. Results keep the
  same provider order, so ranking does not change.
- An expired Switch index is served from cache immediately and refreshed by a detached
  background process. Only a missing index, `--index --refresh` or `--refresh` builds it
  inline.
- Parallel chunked downloads and stall-reconnect logic are not part of this change. The
  measurements showed no per-connection throttle and no connection that stayed slow.

## Capabilities

### New Capabilities
- `provider-search`: how a search fans out to the providers of a platform. This covers
  concurrency and result ordering.

### Modified Capabilities
- `switch-downloads`: the "Archive.org Switch file index" requirement changes. An
  expired index is served while it refreshes in the background, instead of being
  rebuilt inline.

## Impact

- `bin/rom`:
  - `search()` uses a thread pool for each attempt.
  - `ArchiveOrg.switch_index()` serves a stale index and spawns a refresh.
  - A marker file next to `switchindex.json` prevents duplicate refreshes.
- The refresh reuses the existing `switch_index(force=True)` code path in a child Python
  process. No new dependencies.
- Update `README.md` and `AGENTS.md` to describe the cache and concurrency behavior.
