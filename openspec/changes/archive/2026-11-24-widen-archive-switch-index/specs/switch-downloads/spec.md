# Spec Delta

## MODIFIED Requirements

### Requirement: Archive.org Switch file index
`rom` SHALL search the Switch package files (`.nsp`, `.nsz`, `.xci`) of public Archive.org items, including packages stored inside `.zip` archives. It SHALL cover software and data items and items named by a Switch Title ID. It SHALL cache the index for 7 days, and when the cache is older it SHALL search the cached index and refresh it in the background instead of rebuilding inline.

#### Scenario: Game inside a multi-game item
- **WHEN** the user runs `rom --list -p switch "hollow knight"`
- **THEN** the results include the Hollow Knight package file from a public multi-game item

#### Scenario: Index refresh
- **WHEN** the user runs `rom --index --refresh`
- **THEN** the Switch file index is rebuilt and its file count is printed

#### Scenario: Expired index
- **WHEN** the cached Switch index is older than 7 days and the user runs a Switch search
- **THEN** results come from the cached index within a few seconds, and a background refresh replaces the cache file when it completes

#### Scenario: No index yet
- **WHEN** no Switch index cache exists and the user runs a Switch search
- **THEN** the index is built before results are shown

#### Scenario: Refresh already running
- **WHEN** a background refresh started less than 15 minutes ago and another Switch search runs on an expired index
- **THEN** no second refresh is started

#### Scenario: Package inside a zip in a data item named by Title ID
- **WHEN** the user runs `rom --list -p switch "hyrule warriors definitive edition"` and the game exists only as `[0100AE00096EA000].zip` in the data item `0100-ae-00096-ea-000`
- **THEN** the results include Hyrule Warriors: Definitive Edition from archive.org with its `.xci` base and `Update 1.0.1` as choices

#### Scenario: Downloading a zip member
- **WHEN** the user downloads the update choice of that result
- **THEN** only the `(Update 1.0.1).nsp` member is downloaded, not the whole 13.9 GB zip

## ADDED Requirements

### Requirement: Non-Switch Archive.org hits are excluded
Archive.org item-search results on the Switch platform SHALL be dropped when the item has no Switch package or archive, or when its title marks it as a PC port or recompilation.

#### Scenario: PC recompilation
- **WHEN** a Switch search matches the item "Hyrule Warriors: Definitive Edition (Nintendo Switch) – Recompilación para PC", which only holds a `.7z` PC build
- **THEN** that item is not shown as a Switch result
