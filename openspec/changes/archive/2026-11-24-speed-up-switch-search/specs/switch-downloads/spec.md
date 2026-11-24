# Spec Delta

## MODIFIED Requirements

### Requirement: Archive.org Switch file index
`rom` SHALL search the Switch package files (`.nsp`, `.nsz`, `.xci`) inside public multi-game Archive.org items by filename. It SHALL cache that index for 7 days. When the cached index is older than that, `rom` SHALL search the cached index and refresh it in the background instead of rebuilding it inline.

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
