# Changelog

All notable changes to this project will be documented in this file.

## [0.0.8] 2025-12-15

### Added
- Developer Mode (Demo Mode)
- Web UI
  - VMs tab:
    - Filtering by namespace
    - More VM details
    - Sorting by column
    - Search bar
  - Nodes tab:
    - Search bar (still need to adjust the counter)
- Health endpoint

### Changed
- Replaced probes on deployment file (from `/login` to `/healthz`)

### Fixed
- Incorrect CPU model detection on some VMs

## [0.0.9] 2025-12-16

### Added
- Web UI
  - Nodes tab:
    - Sorting by column
  - Snapshots tab:
    - Filtering by namespace
    - Sorting by column

### Fixed
  - Nodes tab:
    - Search counter