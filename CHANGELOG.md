# Changelog

All notable changes to this project will be documented in this file.

## [0.11.1] 2026-07-30

### Fixed
- Topology: the overview colours each node using the same usage thresholds as the Nodes tab, so a node that is green on the Nodes tab is no longer shown amber in the topology view.

### Security
- Updated a bundled library to include an upstream security fix.

## [0.11.0] 2026-07-29

### Added
- Topology: drilling into a storage class, network, or NAD with many items now groups them by namespace, so you can open one namespace at a time. This replaces the "show anyway" prompt.

### Changed
- Topology: the overview now colours each node by its CPU and memory usage (the higher of the two) instead of by VM count, matching the Nodes tab. Nodes with no metrics are shown in grey, and hovering a node shows its CPU and memory figures.
- The cluster footprint panel now shows when live metrics were last read.

## [0.10.0] 2026-07-28

### Added
- Cluster footprint panel: a new view in the About dialog shows how light OVTools is on the cluster — how often it refreshes the inventory, how much of the live metrics it serves from its own cache, and how fresh the data is. It reads what OVTools already tracks, so opening it adds no load.
- Topology: the network view now groups attachments into per-network hubs you can click to open, the same way the storage and namespace views already work, so a busy network no longer draws as one unreadable graph.

### Changed
- The Nodes tab now judges CPU overcommit by real contention instead of the raw ratio. A node is highlighted only when its VMs are actually waiting for CPU time, so a high ratio on an idle node no longer looks like a problem, and genuine contention on a modest ratio is not missed. Hovering a node explains the reading. This reads the signal the cluster descheduler provides; without the descheduler, the previous ratio-based view is kept, so nothing is lost.
- The Health tab flags CPU overcommit the same way — on real contention rather than the ratio — and its CPU and memory overcommit checks now link to background reading.

## [0.9.1] 2026-07-24

### Added
- OVTools now compresses its responses, so pages and tables load faster over the network. It is on by default and can be turned off when a proxy or load balancer in front already compresses.

## [0.9.0] 2026-07-24

### Added
- The Node column now links to the Nodes tab from the VMs, CPU, Disk and Network tabs.
- The VM name in the Disk, Network, CPU, PVC and Snapshot tabs is now a link that opens the VMs tab filtered to that VM.
- The VM details panel shows how the vCPUs are arranged, for example "2 sockets × 2 cores".
- A new uninstall script (uninstall.sh) removes OVTools from a cluster cleanly.
- Topology: a view that is too large to draw now offers a way to narrow it down, either starting from the Overview or stepping back to a smaller group, instead of only a "show anyway" prompt. The Node and NAD views are now protected the same way the other large views are.

### Changed
- Topology: a storage class or namespace with a very large number of volumes now draws a trimmed view with a "+N more" marker instead of an unreadable graph.
- Live usage metrics (CPU, memory, network, disk) are held for a few seconds, so busy screens and the automatic refresh stop re-querying the monitoring stack on every redraw. It can be tuned or turned off with a setting.
- OVTools opens faster on large clusters. It no longer contacts every VM's guest agent for information the cluster already reports.
- The Health tab lists each orphaned disk by name, with why it is orphaned and how to reclaim it, instead of a single count.
- Red status badges (Orphan, Stopped, Lost, and similar) are easier to read against the dark background.

### Fixed
- The Nodes tab search keeps its filter through the automatic refresh and when sorting a column. It used to reset.
- A disk from a migration that has not finished is no longer reported as an orphan that is safe to delete; it is flagged for review instead.
- Health: the IOThreads and virtio-rng suggestions now name the VMs they apply to, instead of showing only a count.

## [0.8.0] 2026-07-14

### Added
- Fleet: a search box finds a VM by name. The groups that match open with their VMs already listed.
- Fleet: the VM details show the last live migration, with its date, the nodes the VM moved between and how long it took.
- The installer adds an OVTools entry to the OpenShift console launcher on clusters that run Virtualization.

### Fixed
- Fleet: the availability dial shows a loading state while it is being calculated. On a large cluster it could stay missing for several minutes.
- Health: the IOThreads suggestion now only appears for VMs that actually do heavy disk work. It used to appear on almost every cluster.
- Health: the virtio-rng suggestion no longer counts a Windows VM without a guest agent as a Linux one.
- Demo mode: the Disks, CPU and storage views now match the VM list. The storage topology no longer draws VMs that do not exist.

## [0.7.2] 2026-07-12

### Fixed
- Fleet: VMs without a hostname now show their internal cluster DNS name.

## [0.7.1] 2026-07-12

### Added
- Fleet: the VM details show a Source: the template the VM was created from, or "Migrated from VMware" for VMs imported by the Migration Toolkit for Virtualization.

### Fixed
- Fleet: the Eviction strategy shows the cluster-wide default when a VM doesn't set one.
- Fleet: the hostname falls back to the VM's configured value when none is reported.
- Fleet: the Activity header no longer overlaps the feed text as it scrolls.
- Topology: the Network view asks before drawing a very large shared network, and keeps the loading indicator visible while it renders.

## [0.7.0] 2026-07-12

### Added
- Fleet: the selected VM shows live dials for CPU, memory, storage, network and disk I/O, plus a 30-day availability dial.
- Fleet: the VM details list its network interfaces, storage volumes and snapshots, and show how the vCPUs are arranged into sockets and cores.
- Fleet: the tree and activity panes can be resized and collapsed, and the tab reopens on the VM you last looked at.
- Topology: double-clicking an object opens its tab filtered to that item.

### Changed
- Topology: each object is drawn as an icon instead of a plain shape.
- Fleet VM details are grouped into a Hardware and a Configuration column.

### Fixed
- Topology: clicking a VM in the Network view shows its IPs, and the Storage and Namespace views show its disks.
- Searching the Network, Nodes and Storage-class tabs now matches the network, NAD, node and storage-class names.
- Double-clicking a node in the topology no longer does nothing or reports an error.
- The VMs and Nodes tables no longer leave empty space on the right when their columns scroll sideways.
- After an upgrade the interface loads the new files, not a cached copy.

## [0.6.1] 2026-07-12

### Fixed
- vCPU counts were wrong for some VMs: those created from the OpenShift console could show 0, and stopped VMs showed 1. They are now read from the VM's own socket and core settings.
- Volumes that had not finished binding showed their size as "0 B" instead of the requested size.
- A VM's primary IP could be listed twice, or be blank when the VM's main connection was on a secondary interface.
- On the Network tab, the Type column was blank for macvtap and binding-plugin interfaces such as passt, and the IPv6 column could show a link-local address in place of the routable one.
- On the Nodes tab, the CPU Overcommit ratio was understated on nodes that reserve part of a CPU for the system, and a taint with no value showed a stray "=" (for example "node-role.kubernetes.io/control-plane=:NoSchedule").
- On the Storage tab, a PVC whose name matched more than one category (such as "redis") could be labeled differently on each refresh. Disks from migrated VMs, including Windows state volumes, are now identified as VM disks.
- Health now treats two cases as warnings rather than errors: the per-node VM agent being briefly down during a normal OpenShift Virtualization upgrade, and a missing network on a stopped VM.
- A VM whose pod was killed for running out of memory was reported only as "failed"; the Health entry now says it ran out of memory.
- A stopped VM defined with the older on/off setting could show as "Unknown" instead of "Stopped".
- When a resource was deleted down to none, its tab kept showing the old entries until the next full refresh.

## [0.6.0] 2026-07-11

### Added
- Fleet tab: browse VMs grouped by namespace or by node. Selecting a VM shows its details next to its recent activity (live migrations, snapshots, creation and last start) and a link to its console.
- Per-VM availability: how much of the recent window (up to 30 days) a VM was running, shown in the VM details, the Fleet tab, and the spreadsheet export. It comes from Prometheus, so it depends on your metrics retention.

### Changed
- The Storage and Namespace topology views drew every PVC at once, which was slow on large clusters. They now open as an overview (one node per storage class or namespace, with a count) that you click to drill into.
- Updated the bundled spreadsheet-export and Kubernetes client libraries.

## [0.5.2] 2026-07-09

### Added
- ovtools now shows when a newer version is available. A mark appears on the footer version, and the About panel gets a line linking to the release. The check runs in your browser, anonymously, once a day, and can be turned off with `OVTOOLS_DISABLE_UPDATE_CHECK`.
- The Nodes tab explains the CPU and Memory Overcommit columns on hover, telling you whether a ratio is within the normal range, and colors the VRAM Total column by the node's real memory usage.

### Fixed
- The "Show anyway" button on very large topology views did nothing when clicked. It now loads the view and shows progress while it draws.
- Disks belonging to migrated VMs showed as "App Data" on the PVC tab instead of "VM Disk". They are now identified as VM disks even when the usual KubeVirt labels are missing.
- The failed-migration entry on the Health tab now shows the date it happened, so a recent failure reads differently from an old one.

### Changed
- The dashboard's "Nodes Requiring Attention" card treated normal CPU allocation as load, so healthy nodes showed up red. It now judges CPU by real usage and memory by how much of it is committed, with both figures and the free memory shown on hover.
- The "Node Distribution by Load" card counted nodes by allocation rather than real load, so on a virtualization cluster nearly every node fell in the top band. The card now counts by real CPU usage.

### Improved
- On the Health tab, the IOThreads and virtio-rng tuning notes were one row per VM and filled the list on large clusters. Each is now a single summary line.

## [0.5.1] 2026-07-04

### Added
- Health now catches many more problems that can take a VM down: VMs running but not ready or with a lost disk; VMs that can't schedule, can't pull their image, or keep crashing; failed disk imports and lost or unbound storage; nodes under memory, disk or PID pressure or with a broken network; node network settings that didn't apply; degraded OpenShift Virtualization, KubeVirt, CDI, SSP or networking operators; failed or killed VM pods; volumes that won't attach; VMs wired to a network that no longer exists; failed live migrations; and the per-node KubeVirt agent missing from a node. Health can be filtered by any of these types.
- Sortable metric columns: CPU, memory and network on the VMs and Nodes tabs, Read and Write IOPS on Disks, and type on Datastores.
- Topology: opening a very large "Storage → PVC → VM" or "Namespace → PVC → VM" view now asks for confirmation first, with a one-time note explaining the view. Popups show a VM's volumes in the Storage view and its network attachments in the NAD view, and busy views group the remainder under "Other".

### Fixed
- Auto-refresh no longer resets your search, filters, sorting or current page.
- The "Snapshots By Age" card on the dashboard was empty; it now populates.
- Health no longer reports a missing eviction strategy, CPU model or resource limits when the VM inherits them from cluster or namespace defaults. That was firing on almost every VM.

### Improved
- Reordered the Topology view dropdown so the busiest view (Network → VMs) comes last.

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
