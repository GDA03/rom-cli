# switch-downloads Specification

## Purpose
Defines how `rom` finds Nintendo Switch titles and downloads them directly from providers
reachable without a browser, captcha, or account, and what it does when no such path exists.

## Requirements

### Requirement: Switch search merges direct-download providers
A Switch search (`-p switch`, or a search across all platforms) SHALL query romslab, the Archive.org
Switch file index, Archive.org item search and nswgame, and merge their results into one ranked list.

#### Scenario: Title found on romslab
- **WHEN** the user runs `rom --list -p switch "mario kart 8 deluxe"`
- **THEN** the results include a romslab entry for Mario Kart 8 Deluxe with platform `Switch`

#### Scenario: Provider unavailable
- **WHEN** one Switch provider times out or returns an error during a search
- **THEN** the results from the remaining providers are still shown and the command does not fail

### Requirement: Direct sources rank above manual-link sources
Results whose source can be downloaded directly (romslab, Archive.org) SHALL rank above nswgame results for an equally matching title.

#### Scenario: Same title on romslab and nswgame
- **WHEN** a Switch search matches the same title on romslab and on nswgame
- **THEN** the romslab result is listed before the nswgame result

### Requirement: romslab downloads use the filekeeper mirror
Downloading a romslab result SHALL fetch the base-game file through the post's filekeeper mirror, without a browser, login or captcha.

#### Scenario: Post with a filekeeper mirror
- **WHEN** the user downloads a romslab result whose post has a filekeeper mirror
- **THEN** the file is saved under `<out>/Switch/` and is validated before being kept

#### Scenario: Post without a filekeeper mirror
- **WHEN** the user downloads a romslab result whose post only links captcha-protected hosts
- **THEN** the download fails with a message that lists the post's host links

### Requirement: Archive.org Switch file index
`rom` SHALL search the Switch package files (`.nsp`, `.nsz`, `.xci`) inside public multi-game Archive.org items by filename. It SHALL cache that index for 7 days.

#### Scenario: Game inside a multi-game item
- **WHEN** the user runs `rom --list -p switch "hollow knight"`
- **THEN** the results include the Hollow Knight package file from a public multi-game item

#### Scenario: Index refresh
- **WHEN** the user runs `rom --index --refresh`
- **THEN** the Switch file index is rebuilt and its file count is printed

### Requirement: Restricted Archive.org items are excluded
The Archive.org Switch index SHALL exclude items that need an account (`loggedin` collection, `access-restricted-item`, or private files).

#### Scenario: Restricted item
- **WHEN** a large Switch item on Archive.org is login-restricted
- **THEN** none of its files appear in Switch search results

### Requirement: Base game preferred within an item
When an Archive.org item holds several files for the same title, the base game SHALL be offered ahead of updates and DLC.

#### Scenario: Base, update and DLC in one item
- **WHEN** a search matches a base `.nsp`, an update `.nsp` and a DLC `.nsp` of one title
- **THEN** the base file ranks first and each result shows its type (Base, Update or DLC)

### Requirement: Pixeldrain and mediafire links download directly
Downloading an nswgame or dlpsgame result SHALL fetch the file directly when the post links pixeldrain or mediafire. It SHALL try those hosts before hosts that cannot be fetched directly.

#### Scenario: Post links pixeldrain
- **WHEN** the user downloads an nswgame result whose post links a pixeldrain file
- **THEN** the file is downloaded without user interaction

### Requirement: Archives are kept as downloaded
`rom` SHALL save `.rar`, `.7z` and `.zip` downloads unchanged, without extracting them. A `.rar` download SHALL be valid only when it starts with the RAR signature.

#### Scenario: RAR download
- **WHEN** a Switch download delivers a RAR archive
- **THEN** it is saved with a `.rar` extension and is not extracted

#### Scenario: Error page instead of archive
- **WHEN** a host returns an HTML page instead of the expected RAR archive
- **THEN** the file is deleted and the download is reported as failed

### Requirement: Manual-link fallback
When no linked host of a result can be fetched directly, the download SHALL fail with a message that names the host links. `--print-url` SHALL print those links.

#### Scenario: Only captcha hosts
- **WHEN** the user downloads an nswgame result that only links captcha or Cloudflare-protected hosts
- **THEN** the command reports the failure and prints up to four of the links to open in a browser
