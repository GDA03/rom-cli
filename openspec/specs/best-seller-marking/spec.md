# best-seller-marking Specification

## Purpose
Decides which search and browse results count as Wikipedia best-sellers, which drives both the ★
marker and the best-seller ranking boost in the CLI and the TUI.

## Requirements

### Requirement: Best-seller match is exact
A result SHALL count as a best-seller only when its normalized title equals a normalized title from that platform's best-seller list.

#### Scenario: Exact title
- **WHEN** a GBA result is titled "Mario Kart: Super Circuit" and the GBA best-seller list contains "Mario Kart: Super Circuit"
- **THEN** the result is marked ★

#### Scenario: Best-seller title is only a prefix
- **WHEN** an N64 result is titled "Mario Kart 64: Chaos Blast" and the list contains "Mario Kart 64"
- **THEN** the result is not marked ★

### Requirement: Hacks are never best-sellers
A result whose title contains a hack, romhack, translation, beta, prototype or demo marker SHALL NOT count as a best-seller, even when its title matches the list.

#### Scenario: Hack with a matching title
- **WHEN** a result titled "Super Mario Kart (Hack)" is returned for SNES
- **THEN** the result is not marked ★ and gets no best-seller ranking boost
