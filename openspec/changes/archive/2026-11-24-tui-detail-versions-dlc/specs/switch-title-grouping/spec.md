# Spec Delta

## Purpose

Groups the package files of one Nintendo Switch game (base, updates, DLC) by Title ID so they can be
shown, chosen and downloaded together.

## ADDED Requirements

### Requirement: Files are grouped by base Title ID
Switch package files whose names carry a 16-digit Title ID SHALL be grouped under the base Title ID. The base ID is the Title ID with its update and DLC bits cleared: suffix `000` is the base, `800` an update, and any other suffix a DLC.

#### Scenario: Base, update and DLC of one game
- **WHEN** the index holds `Game [01004EA01CED8000][v0].nsp`, `Game [01004EA01CED8800][v65536].nsp` and `Game DLC [01004EA01CED9001][v0].nsp`
- **THEN** all three files belong to the group `01004EA01CED8000` as Base, Update and DLC

#### Scenario: Non-Latin titles
- **WHEN** files of one game have Japanese titles that differ between the base and its DLC
- **THEN** they are still grouped together by Title ID

### Requirement: One result row per Switch game
A Switch search SHALL show one result row per grouped game. The row SHALL summarize how many updates and DLC are available.

#### Scenario: Game with extras
- **WHEN** a search matches a game that has a base file, two updates and one DLC
- **THEN** a single row is shown for the game with the summary "+2 upd · 1 DLC"

#### Scenario: Extras without a base
- **WHEN** a search matches a game whose group has updates but no base file
- **THEN** the row is shown and marks the base as unavailable

### Requirement: Files without a Title ID stay separate
Switch package files without a Title ID in their name SHALL remain individual result rows and SHALL NOT be merged by name similarity.

#### Scenario: Name-only file
- **WHEN** an indexed file named `Celeste (NSP)(eShop).rar` has no Title ID
- **THEN** it appears as its own result row

### Requirement: Updates ordered newest first
Updates in a group SHALL be ordered by release number, newest first. The release number is the bracketed `vN` value divided by 65536. A human-readable version such as `v1.2.1` in the name SHALL be shown when present.

#### Scenario: Two update releases
- **WHEN** a group has updates tagged `[v1245184]` and `[v1507328]`
- **THEN** the `[v1507328]` update, release 23, is listed before the `[v1245184]` update, release 19
