# tui Specification

## Purpose
Defines how the interactive terminal UI shows search and browse results and how its keys behave,
so users can tell results apart, select them, and act on them predictably.

## Requirements

### Requirement: Download path marker
Each result row SHALL show whether its source downloads directly or only yields host links to open in a browser.

#### Scenario: Direct and link-only sources
- **WHEN** a search returns a romsfun result and an nswgame result
- **THEN** the romsfun row shows the direct-download marker and the nswgame row shows the link-only marker

### Requirement: Empty columns are hidden
The Size and Type columns SHALL be hidden when no displayed row has a value for them, and the Title column SHALL use the freed width.

#### Scenario: Retro search without sizes
- **WHEN** every displayed row has an unknown size and no type
- **THEN** the table shows no Size or Type column

#### Scenario: Switch search with types
- **WHEN** at least one displayed row has a type such as Base or Update
- **THEN** the Type column is shown

### Requirement: Marked rows stay readable
A marked row SHALL keep its text legible and show a visible check mark in the mark column.

#### Scenario: Marking a row
- **WHEN** the user presses space on a row
- **THEN** the row shows a check mark and its title remains readable against the mark highlight

### Requirement: English labels
The platform picker and the status line SHALL label the all-platforms option "All".

#### Scenario: Opening the picker
- **WHEN** the platform picker opens
- **THEN** its first option reads "All"

### Requirement: Enter downloads the highlighted row
Pressing Enter on a result row SHALL download that row without changing the set of marked rows. The options palette SHALL stay available on ctrl+b.

#### Scenario: Enter with nothing marked
- **WHEN** the user presses Enter on a row while no rows are marked
- **THEN** that row's download starts and no row becomes marked

#### Scenario: Enter with other rows marked
- **WHEN** two other rows are marked and the user presses Enter on a third row
- **THEN** only the third row is downloaded and the two marks remain

### Requirement: Platform switch browses when the search field is empty
After a platform is chosen, the TUI SHALL browse that platform's catalog when the search field is empty, and SHALL re-run the search on the new platform when the field has text.

#### Scenario: Empty field
- **WHEN** the search field is empty and the user picks PS5 in the platform picker
- **THEN** the PS5 catalog is listed

#### Scenario: Field has a query
- **WHEN** the search field contains "zelda" and the user picks Switch
- **THEN** "zelda" is searched on Switch

### Requirement: Search field shortcut
Pressing `/` SHALL move focus to the search field from the result table.

#### Scenario: Searching again
- **WHEN** the result table has focus and the user presses `/`
- **THEN** the search field has focus and typing goes into it

### Requirement: Key hints live in the footer
Key hints SHALL be shown in the footer. The status line SHALL carry only messages, such as result counts, progress and errors, and SHALL fit on one line at 120 columns.

#### Scenario: Rows marked at 120 columns
- **WHEN** two rows are marked in a 120-column terminal
- **THEN** the status line shows the result and selection counts on a single line and the footer lists the key hints

### Requirement: Truncated results are announced
When a search returns more rows than the table displays, the status line SHALL state how many are shown out of the total.

#### Scenario: More than 200 results
- **WHEN** a search returns 412 results
- **THEN** the status line reads "200 of 412 results"

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
