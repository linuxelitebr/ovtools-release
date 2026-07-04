# Changelog

All notable changes to this project will be documented in this file.

## [0.5.0] 2026-07-03

### Added
- CSI VolumeSnapshots are now first-class. A single Snapshots tab shows every VolumeSnapshot with its readiness, source PVC, age and owning VM, links VMSnapshot-managed rows back to their VMSnapshot, and flags snapshots set to retain their content. Snapshot classes are shown for reference, and orphaned snapshot content (storage left behind with no VolumeSnapshot pointing at it) is surfaced as a Health check you can drill into.
- Large tables now load in pages. VMs, Snapshots, PVCs, Disks, Network, CPU and Snapshot Contents paginate (50 rows per page by default, configurable and remembered in the browser), so clusters with thousands of objects stay fast. Search, filters and sorting carry across pages.
- Two new topology views. "NAD to VMs" groups virtual machines under the network attachment definition they connect to, replacing the single crowded multus hub. "Namespace to PVC to VM" groups persistent volume claims under their namespace, spreading storage out instead of piling it onto one storage-class hub. The older views remain in the dropdown.
- The Health tab can be filtered by resource type, making it easy to isolate orphans or any other kind of check. It works alongside the existing severity and namespace filters and the drill-down links.

### Fixed
- Switching tabs while auto-refresh was running could throw a "Canvas is already in use" error on the dashboard. Fixed.
- Search on the VMs tab now filters reliably from the server instead of silently doing nothing.

### Improved
- Status badges are consistent everywhere now: uppercase, no leading dot, and a single smaller text size across all tabs (including Ready / NotReady on Snapshots and unified Health severities).
- Exported spreadsheets write true/false in English instead of a locale word that could overflow the column.

## [0.4.0] 2026-05-23

### Added
- Ceph health card on the Health tab. Auto-detected. If Prometheus is federating the Ceph metrics (HEALTH_OK / WARN / ERR, OSD status, PG degradation, recent crashes, near-full pools), every firing health check shows up as a row with severity, plain-English explanation and a link to the upstream docs anchor. Nothing shows up on clusters without Ceph; no flag, no extra config.
- Tri-state Prometheus connection indicator (green, yellow, red). Green is data flowing. Yellow is the half-failure case: Prometheus answers HTTP but the kubevirt or node-exporter scrape pool dropped, so every metric column would render as `-`. Red is the endpoint not answering at all. The previous indicator only had green or red, so a degraded Prometheus showed up as "connected" while the dashboard quietly emptied out.
- Banner above the VMs and Nodes tabs when Prometheus is reachable but returning no scrape data. Spells out the likely cause (degraded `prometheus-k8s` pod, missing ServiceMonitor) so the operator knows where to look instead of blaming ovtools.
- Top dashboard cards are now clickable. Total VMs, Running, Stopped, vCPUs Allocated, Nodes, Old Snapshots, Health Issues each navigate to the matching tab with the right filter pre-applied. Keyboard-accessible (`Tab` + `Enter`).
- Persistent banner when the connected cluster has no KubeVirt installed. Explains that ovtools is built for OpenShift Virtualization and most tabs will be empty otherwise. Dismissal is per-cluster, so connecting to a different cluster without KubeVirt brings it back.
- Tooltip on the Datastores tab explaining why Used % can exceed 100 % when two StorageClasses share the same Ceph pool. Lists which other StorageClasses share the pool so the user can do the math.

### Fixed
- Column chooser silently broke after revisiting a tab. The inline `<script>` re-declared a top-level `const`, threw SyntaxError on every subsequent load, and the rest of the script never ran. Symptom: clicking a checkbox rerouted the page to the VMs tab. Tab templates now wrap their scripts in IIFEs.
- Bar clicks on "VMs per Node", "vCPU Allocation per Node" and "Memory Allocation per Node" routed to an empty VMs tab because the node label was truncated for the chart axis and then reused as the filter target. The click handler now sends the full FQDN.
- VM disks created from instance-type DataSources were misclassified as "Template" on the PVCs tab. The CDI controller propagates the template label onto every cloned disk, so the heuristic was catching every running VM's root disk. Fixed to only count canonical template PVCs.
- Topology view switcher crashed with `TypeError: data.edges is null` on clusters with no matching resources. The Go handlers returned nil slices that JSON encoded as `null`. They now always return arrays.
- Cytoscape threw `renderer is null` after navigating away from the Topology tab because a queued resize callback fired on a destroyed instance. The callback now checks `cy.destroyed()` before touching the renderer.
- Chart.js threw `Canvas is already in use` after coming back to the Dashboard. Orphaned chart instances from earlier renders stayed in the registry and conflicted with the new canvases. The dashboard now sweeps every live Chart.js instance before creating new ones.
- Tab swaps flashed the loading overlay for a single frame on every cached response. The cache serves a tab in roughly 5 ms; the overlay went on and off inside the same paint window, which the eye reads as flicker. The spinner is now deferred and only paints if the load actually takes longer than 200 ms.

### Improved
- Auto-refresh on tabs with thousands of rows no longer rebuilds the entire DOM every 60 seconds. The refresh path uses morph; only changed cells get updated, the rest stay in place. Tab switches still use a clean swap so listeners do not leak between tabs.
- Stale-while-revalidate is now formally documented in CLAUDE.md as a non-negotiable, with tests that fail if a handler bypasses the cache or talks to Prometheus or the k8s API on the render path.
- Most of the per-tab UI plumbing moved into shared static modules (`col-chooser.js`, `row-count.js`). Each tab template now declares its configuration via `data-*` attributes instead of carrying its own 80-line copy of the same JS. The browser caches one module instead of re-parsing inline blocks on every swap.

### Removed
- Per-PVC Block usage via `rbd du`. The original v0.4.0 plan called for filling in the `-` placeholder on Block-mode PVCs by talking to the Ceph MGR. After verifying against ODF 4.21, the only path that does not require either a CLI flag (rejected) or a librados dependency on the ovtools side was to expose `ceph_rbd_image_used_bytes` from the MGR Prometheus module, which ODF does not emit. The placeholder stays. Documented in `docs/ENTERPRISE_ROADMAP.md` along with the conditions under which the item could come back.

## [0.3.3] 2026-05-16

### Added
- Privacy notice popup on first open of the dashboard, with a 10-second auto-dismiss timer and a re-open link from the About dialog
- `PRIVACY.md` at the repo root documenting what ovtools processes, where the data lives, what it never does, what the logs contain, and how LGPD/GDPR legal basis sits with the operator
- Community files: `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, `THIRD_PARTY_LICENSES.md`, issue and pull-request templates

### Fixed
- Bumped `golang.org/x/net` to close GO-2026-4918, an HTTP/2 transport infinite loop reachable through the login flow
- Mock client now flags Block PVCs so the Datastores tooltip is visible in developer mode

### Improved
- CI runs `govulncheck` on every push and pull request, catching Go CVEs before Dependabot picks them up
- Container image scanned by Trivy on each release; high and critical findings fail the build
- SPDX SBOM emitted by `anchore/sbom-action` and attached to the GitHub Release
- README gets the standard set of badges (CI status, latest release, license, Go version, GHCR image), repo topics added for discoverability
- `.dockerignore` cleaned up and a stray customer-name reference removed
- ENTERPRISE_ROADMAP reframed around v1.0.0 as the technical milestone where enterprise features land, without pricing-tier language

## [0.3.2] 2026-05-06

### Changed
- Project moved to the `elastocera` GitHub org. The container image now lives at `ghcr.io/elastocera/ovtools` and the repository URL is `github.com/elastocera/ovtools`. The public `linuxelitebr/ovtools-release` mirror still gets release artifacts during the transition.
- Toast notifications redesigned to match the dashboard cards: subtle top accent instead of a thick coloured left border, plus a proper error variant.

### Fixed
- Container images now report the version they were built with. Earlier builds always reported the version baked into source, regardless of which tag was used.

## [0.3.1] 2026-04-10

### Added
- New `-prometheus-url` flag (and `OVTOOLS_PROMETHEUS_URL` env var) to override the auto-discovered Prometheus endpoint, useful when running the binary outside the cluster (e.g. via `oc port-forward`)

### Improved
- `install.sh` now detects upgrade conflicts on the Deployment selector and offers to recreate the Deployment automatically, preserving all other resources

## [0.3.0] 2026-04-09

### Added
- VM detail drawer: click any VM row to see its specs, disks and network interfaces side-by-side without leaving the table
- Info tooltip on Datastores explaining why Block mode volumes show "-" for usage, with a link to the Kubernetes documentation

### Improved
- Dashboard redesigned with clean typography and contextual icons; alerts now surface through a subtle top accent instead of a heavy colored bar

## [0.2.2] 2026-04-08

### Fixed
- Datastores showing near-zero or incorrect usage for storage classes with Block mode PVCs (VM disks)
- Datastores showing pool-level usage for Filesystem-only storage classes that share a Ceph pool
- Shows "-" instead of misleading "0 B" when no usage data is available for Block volumes
- "Initial data load complete" notification disappearing before user could read it on large clusters
- Release workflow now includes `deployment-legacy.yaml` in assets and uses `.zip` for Windows binary

### Improved
- Release notes on GitHub now show only the current version instead of the full changelog
- OpenShift deployment YAMLs now use `imagePullPolicy: IfNotPresent` for the ovtools container

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

## [0.0.1 - 0.0.7] 2025-10 to 2025-12-14

The earliest scaffolding of the project. Detailed per-version notes from that period are not preserved, so this entry summarises what the codebase looked like by the time 0.0.8 landed:

### Added
- Go web server (`cmd/ovtools/main.go`) with a single `internal/server` package serving HTML templates plus htmx for partial swaps.
- Manual login screen accepting either a bearer token (paired with the API server URL) or a pasted kubeconfig. Session lived in memory and expired after one hour.
- In-memory cache wrapping the Kubernetes client so the dashboard did not refetch on every request.
- First read-only inventory views: a VMs tab listing virtual machines with their basic spec, and a Nodes tab listing cluster nodes.
- A Dashboard tab with summary counts (total VMs, running, stopped, vCPUs allocated, nodes ready) rendered as cards.
