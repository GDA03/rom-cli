# Spec Delta

## ADDED Requirements

### Requirement: Detail panel
The TUI SHALL open a detail panel for the highlighted result on a dedicated key. The panel SHALL show the result's source, its file name or names, whether it downloads directly, and every host link as selectable text.

#### Scenario: Link-only result
- **WHEN** the user opens the detail panel of an nswgame result
- **THEN** the panel lists the post's host links grouped by Base, Update and DLC, and the links can be copied

#### Scenario: Closing the panel
- **WHEN** the detail panel is open and the user presses escape
- **THEN** the panel closes and the highlighted row is unchanged

### Requirement: Version choice in the detail panel
The detail panel SHALL present the result's versions as one single-choice base or file selection and, for Switch, multi-choice Update and DLC selections. Confirming SHALL queue the chosen files.

#### Scenario: Queue from the panel
- **WHEN** the user picks a base version, ticks an update and confirms
- **THEN** both files are added to the download queue

### Requirement: Download queue panel
Downloads SHALL be shown in a queue panel. The panel SHALL show one entry per file with its own progress, keep each failure message visible, and let the user cancel a queued or running item.

#### Scenario: One item fails
- **WHEN** the second of three queued downloads fails
- **THEN** its error stays visible in the queue while the third download proceeds

#### Scenario: Cancelling a running download
- **WHEN** the user cancels a running download
- **THEN** the transfer stops, its partial file is kept for resume, and the entry shows "cancelled"

### Requirement: Duplicate titles are merged
Results with the same normalized title and platform from different sources SHALL be shown as one row. That row's sources SHALL be selectable in the detail panel. The default source SHALL be the highest-ranked one.

#### Scenario: Same game on romsgames and romsfun
- **WHEN** a search returns "Mario Kart DS" for NDS from both romsgames and romsfun
- **THEN** one "Mario Kart DS" row is shown and its detail panel offers both sources

### Requirement: Filterable platform picker
The platform picker SHALL list Switch and the most-used platforms first, and SHALL filter its options as the user types.

#### Scenario: Typing in the picker
- **WHEN** the picker is open and the user types "ps"
- **THEN** only PS1, PS2, PS3, PS4, PS5, PSP and PS Vita remain listed
