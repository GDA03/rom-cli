# Spec Delta

## Purpose

Lets the user see every version or file available for one result and choose which ones to download,
with defaults that match the automatic choice so non-interactive downloads keep working.

## ADDED Requirements

### Requirement: Versions are listed per result
For a result that has more than one downloadable file, `rom` SHALL list each file with its label and size. For Switch, the label shows the format, region and version; for romsfun, the region, revision or hack tag.

#### Scenario: romsfun game with several files
- **WHEN** the user opens the versions of "Mario Kart: Super Circuit" on romsfun
- **THEN** seven files are listed, including the Europe, USA, Wii U Virtual Console, China (Proto) and hack entries

#### Scenario: Switch game with several base files
- **WHEN** the user opens the versions of Zelda: Tears of the Kingdom
- **THEN** the base files `v0 US .nsp`, `v0 US .xci` and `v1.2.1 US .xci` are listed separately

### Requirement: Default choice matches automatic download
The pre-selected file SHALL be the one `rom` would download without a choice: for romsfun, the USA or World non-hack file; for Switch, the `.nsp` base and the newest update. Downloading without opening the version list SHALL download those defaults.

#### Scenario: Download without choosing
- **WHEN** the user runs `rom -y -p gba "mario kart super circuit"`
- **THEN** the USA No-Intro file is downloaded, as before this change

### Requirement: A chosen version is downloaded
When the user chooses a file, `rom` SHALL download that file instead of the default.

#### Scenario: Choosing a region
- **WHEN** the user picks the Europe file of "Mario Kart: Super Circuit" and downloads
- **THEN** the downloaded file is the Europe release

### Requirement: Switch updates and DLC can be added
For a grouped Switch game, the user SHALL be able to tick any number of updates and DLC in addition to one base file. Each ticked item SHALL be downloaded as a separate file.

#### Scenario: Base plus newest update plus one DLC
- **WHEN** the user keeps the default base, keeps the newest update ticked and ticks one DLC
- **THEN** three files are queued for download

### Requirement: Link-only versions are shown but not queued
Versions available only through link-only hosts SHALL be listed with their type and host, and SHALL NOT be queueable for automatic download.

#### Scenario: romslab update on datanodes
- **WHEN** a romslab post offers an Update only on datanodes
- **THEN** the update is listed as a manual link and cannot be ticked
