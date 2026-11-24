# provider-search Specification

## Purpose
Defines how a search fans out to the providers of a platform so that it waits for the slowest
provider rather than the sum of all of them, while keeping result order stable.

## Requirements

### Requirement: Providers are queried concurrently
Within one query attempt, `rom` SHALL query the providers of the selected platform concurrently. With no platform selected, it SHALL query the cross-platform providers concurrently.

#### Scenario: Several slow providers
- **WHEN** a platform has three providers that each take about one second to answer
- **THEN** the search attempt finishes in about one second, not three

#### Scenario: One provider fails
- **WHEN** one provider raises an error or times out during a concurrent attempt
- **THEN** the results from the other providers are still returned

### Requirement: Result order is independent of response timing
Results SHALL be merged in the providers' configured order, whichever provider answers first, so the same responses always produce the same ranked list.

#### Scenario: Later provider answers first
- **WHEN** the last configured provider responds before the first one
- **THEN** the merged results list the first provider's results before the last provider's
