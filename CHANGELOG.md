# Changelog

All notable changes to this project will be documented in this file.

## [0.2.2] 2026-04-08

### Fixed
- Datastores showing near-zero or incorrect usage for storage classes with Block mode PVCs (VM disks)
- Datastores showing pool-level usage for Filesystem-only storage classes that share a Ceph pool
- Shows "-" instead of misleading "0 B" when no usage data is available for Block volumes
- "Initial data load complete" notification disappearing before user could read it on large clusters
- Release workflow now includes `deployment-legacy.yaml` in assets and uses `.zip` for Windows binary

### Improved
- Release notes on GitHub now show only the current version instead of the full changelog

## [0.2.1] 2026-04-07

### Fixed
- Datastores tab showing incorrect or zero usage for large storage classes 
- Users with group-based permissions (e.g. LDAP) getting "forbidden" errors despite having cluster-admin access 

## [0.2.0] 2026-04-02 - *Easter Bunny Release* 🐰

### Highlights
- In-cluster SSO with OAuth Proxy
  - Automatic login via OpenShift (no manual token needed)
    - Users already logged into the OpenShift console are seamlessly authenticated in ovtools
    - Works on both standalone OpenShift and Hypershift clusters
    - Includes:
      - oauth-proxy sidecar
      - Required RBAC (system:impersonator, Prometheus access). Use `install.sh` and the new `deployment.yaml`
      - Automatic TLS certificate generation
    - Logout now properly terminates both ovtools and OpenShift sessions
> The OAuth Proxy works when ovtools is installed on the cluster and is not run from the binary

### Added
- Health Check
  - New VM tuning checks (especially for VMware-migrated workloads):
    - CPU, NIC, Disk (bus/cache), RNG, IOThreads
  - Detects suboptimal configurations with Warning/Info levels
  - Direct links to remediation documentation
  - RNG check skipped for Windows VMs
  - Search/filter support across checks
- Export (XLSX and CSV)
  - Health: added reference URL column
  - CPU: requests and limits
  - PVC: usage status and DataVolume
  - Nodes: allocatable CPU/memory and kubelet version
  - Snapshots: status and size
  - Disks: cache, volume mode, and hotplug

### Fixed
- Logout behavior fixed with OAuth Proxy (prevents unintended silent re-login)
- Dashboard:
  - Charts no longer break after auto-refresh
  - Fixed missing data issue caused by k8s client race condition
- Topology:
  - UI state now persists across refreshes

### Changed
- Deployment
  - ovtools now binds to 127.0.0.1 when using OAuth Proxy
  - TLS switched to re-encrypt (public cert externally, cluster CA internally)
  - Service port changed to 8443 (proxy)
  - Added deployment-legacy.yaml:
    - For older setups (no OAuth, TLS, or RBAC)
- Web UI
  - Auto-refresh now shows a progress bar and remaining time
  - Tooltips added for long table values
  - Dashboard charts render correctly when switching tabs

## [0.1.9] 2026-03-21

### Added
- Web UI
  - Column Chooser on all remaining tabs: Nodes, Disks, Networks, CPUs, PVCs, Snapshots, Datastores
  - All tabs now have consistent column selection UX with localStorage persistence
- CI/CD
  - GitHub Actions CI pipeline: automated `go build`, `go vet`, `go test` on every push/PR
  - Updated Actions to Node.js 24 compatible versions (actions/checkout@v5, actions/setup-go@v6)
- Testing
  - 153 unit tests covering sort functions, security helpers, and config

### Changed
- Web UI
  - Replaced full-page refresh with idiomorph (DOM morphing): eliminates screen flicker on sorting, filtering and tab switching
  - Topology tab intentionally excluded (will be addressed separately)

### Security
- Fixed potential XSS in display names (`html.EscapeString`)
- Fixed URL injection in console URL construction (`url.PathEscape` + K8s name validation)
- Added security headers middleware (X-Content-Type-Options, X-Frame-Options, Referrer-Policy, Permissions-Policy)

## [0.1.8] 2026-03-15

### Added
- Web UI
  - Column Chooser in vInfo tab: select which columns are visible
  - Column preferences persisted in localStorage across sessions
  - Reset to defaults button in column chooser panel
  - Columns `Created` and `Started` (optional, off by default)
  - VM name as clickable link to OpenShift console (real clusters only)

### Changed
- Web UI
  - vInfo tab: optional columns (Net Rx, Net Tx, Disks, NICs, Agent Version) hidden by default to reduce visual noise
  - Empty state rows use `colspan="99"` to remain correct regardless of visible column count

## [0.1.7] 2026-01-07

### Added
- XLSX Export
  - Summary sheet as first tab with executive overview
  - vCluster sheet with cluster info, totals and ratios
  - vMemory sheet with VM memory details and node context
  - vGuestAgent sheet with guest agent status and OS details
  - vEvents sheet with recent VM-related events (limited to 24h, max 500)
  - vMigration sheet with live migration history
  - vDataVolume sheet with CDI DataVolume details
  - vTemplate sheet with VM templates catalog
  - OVTools version included in Summary sheet
  - Merged title cell in Summary for cleaner presentation
- Web UI
  - Loading indicator on XLSX export button
  - Button disabled during export generation

### Changed
- XLSX Export
  - Sheet order reorganized: Summary > vCluster > technical sheets
  - vInfo enriched with Uptime, Guest Agent data, Labels, Annotations
  - vHost enriched with Taints, Boot Time, Uptime

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

## [0.1.5] 2025-12-19

### Added
- Web UI
  - Disks tab:
    - IOPS metrics

## [0.1.4] 2025-12-19

### Added
- Web UI
  - VM tab:
    - IOPS metrics

## [0.1.3] 2025-12-18

### Fixed
- Web UI
  - VM tab:
    - Some VMs were appearing with unknown status
  - Topology tab
    - The Reset View button is not working

## [0.1.2] 2025-12-18

### Added
- Web UI
  - Topology tab:
    - Network and storage topology

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
