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

## [0.1.0] 2025-12-17

### Added
- Web UI
  - Cluster connection indicator
  - Health tab:
    - Filtering by namespace
    - Sorting by column
    - Filtering by severity
  - Disks tab:
    - Filtering by namespace
    - Sorting by column
    - Search by VM, disk or PVC bar
  - Network tab:
    - Filtering by namespace and NAD
    - Sorting by column
    - Search by VM, MAC and IP bar
  - CPU tab:
    - Filtering by namespace
    - Sorting by column
    - Search by VM, node and model bar
  - PVC tab:
    - Filtering by namespace
    - Sorting by column
    - Search by VM, PVC, App and SC bar
  - Datastore tab:
    - Sorting by column
  - Snapshots tab:
    - Search by VM or snapshot bar
  - Dashboard tab
    - Cards:
      - Node Distribution by Load
      - Nodes Requiring Attention
      - PVCs per Storage Class
      - PVC Status Overview

## [0.1.1] 2025-12-18

### Added
- Web UI
  - Topology
  - Node and Guest Agent filters on VMs tab
  - Age and Ready filters on Snapshots tab
  - Storage Class and PVC Status filters on Disks tab
  - Node filter on Networks tab
  - Storage Class, Status, and Type filters on PVCs tab
  - Node and VM Status filters on CPUs tab
  - Smooth hover transitions on Topology view
  - Filtering for tabs that didn't have them


## [0.1.2] 2025-12-18

### Added
- Web UI
  - Topology tab:
    - Network and storage topology

## [0.1.3] 2025-12-18

### Fixed
- Web UI
  - VM tab:
    - Some VMs were appearing with unknown status
  - Topology tab
    - The Reset View button is not working

## [0.1.4] 2025-12-19

### Added
- Web UI
  - VM tab:
    - IOPS metrics

## [0.1.5] 2025-12-19

### Added
- Web UI
  - Disks tab:
    - IOPS metrics

## [0.1.6] 2025-12-30

### Added
- Web UI
  - General
    - Visual data loading indicator
    - OnClick events to dashboard charts
  - Topology:
    - Overview with drill-down
- Engine
  - Default API request timeout of 60s
  - API timeout adjustment by parameter
  - Optimized data loading and cache usage for large clusters
  - Added adaptive pre-fetch that adjusts refresh intervals to cluster latency
