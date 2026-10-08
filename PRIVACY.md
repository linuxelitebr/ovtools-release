# Privacy

ovtools is an inventory and operational visibility dashboard. The whole point of the tool is to read your cluster state and show it to you in one place. This document describes, in plain language, what happens to that data while you use it.

## What ovtools processes

The tool talks to two endpoints inside your OpenShift cluster:

- **The Kubernetes API server**, to list VirtualMachines, VirtualMachineInstances, Nodes, PVCs, Storage Classes, Snapshots, and the other resources that fill the dashboard.
- **Prometheus / Thanos Querier**, to fetch usage metrics (VM CPU and memory, node utilization, IOPS, Ceph pool usage).

Both endpoints already hold this data. ovtools does not introduce new sources.

## Where the data lives

- **In the server process, in RAM only.** The Go binary keeps an in-memory cache so the dashboard does not refetch on every request. Cache TTL is configurable; default is 60 seconds. The cache is plain Go data structures, never written to disk by ovtools.
- **In your browser, only for the duration of the page.** The HTML the server sends is rendered, displayed, and discarded when you reload or close the tab. ovtools does not write cluster data into `localStorage`, `IndexedDB`, or service-worker caches.
- **In XLSX or CSV exports you generate.** Those files are produced on demand and streamed straight to your browser. ovtools does not retain a copy after the response is sent.

## What ovtools never does

- **No telemetry.** There is no analytics SDK, no crash reporter, no "phone home" ping. The binary has no network egress configured beyond the cluster API server and the cluster's Prometheus route.
- **No third-party fetches.** Every static asset (HTML, CSS, JS, fonts) ships in the binary. Nothing is pulled from a CDN at runtime.
- **No tracking cookies.** The one cookie ovtools sets is a session token used to keep you logged in. It is HttpOnly, scoped to the ovtools origin, and expires when your session ends.
- **No data retention beyond your session.** Sessions expire after one hour of inactivity. When a session ends, the associated cache reference is released; the next request that needs the data refetches from the cluster.

## What the logs contain

ovtools writes operational lines to stdout. This is what your container platform's logging stack captures. By category:

- **Startup and lifecycle:** version banner, cache pre-fetch start and complete (with elapsed time), refresh interval, shutdown.
- **Cache timings:** "refreshing cache (interval was 48s)", "next refresh in 60s". Nothing about the contents.
- **Errors from handlers:** when a fetch from the API server fails, the underlying error is logged. That error can include the cluster API server URL and the resource path (e.g. `/api/v1/namespaces/<namespace>/persistentvolumeclaims`). Namespace names can appear in those paths.
- **Authentication failures:** when impersonation fails for a user, the username and the reason are logged. The token itself is never logged.

The logs do **not** contain VM names, PVC contents, secrets, tokens, kubeconfig data, request headers, or any resource payload. If you need to silence even the operational lines, set your container logging policy to drop them at the platform level. A configurable `-log-level` flag may land in a future release.

## Honest caveats

A few things outside ovtools' control that you should be aware of:

- **Kernel swap.** If the node hosting the ovtools Pod is under memory pressure and the OS swaps the cache pages to disk, those pages live on the swap device for whatever lifetime the kernel decides. This is a Linux kernel behaviour, not an ovtools feature.
- **Browser extensions.** Anything you have installed in the browser tab that shows ovtools (password managers, page-readers, analytics blockers, ad blockers, AI assistants) can read what ovtools renders. ovtools cannot defend against the user's own browser.
- **Container platform logging.** The startup and error lines described above leave ovtools as soon as they are written to stdout. Where they go from there (Loki, Elastic, journald, splunk) is a property of the cluster, not of ovtools.
- **Network captures.** Traffic between ovtools and the API server is TLS-encrypted, but anyone with access to the cluster network or to a mirror port can observe the metadata (which endpoint, how often, payload size). Standard for any in-cluster HTTPS workload.

## Legal basis (LGPD, GDPR, others)

ovtools does not establish a legal basis for processing cluster data on your behalf. That responsibility lies with the operator running ovtools: the person or organisation pointing the tool at a cluster decides what data is in that cluster, who can access ovtools, and under which lawful basis. ovtools is a viewer over data you already have.

For Brazilian LGPD specifically:

- **Controlador** = the operator running ovtools against a real cluster (not ovtools maintainers).
- **Tratamento** = exibir, em RAM, dados que o cluster já contém, para o usuário autenticado.
- **Base legal** = definida pelo controlador (legítimo interesse operacional, contrato, etc.).
- **Retenção** = limitada ao TTL do cache mais a duração da sessão.

If you need a Data Processing Agreement or any other formal artifact, treat ovtools as on-premises software you run, not as a service ovtools maintainers operate on your behalf.

## Verifying the claims here

This is open source. The relevant code:

- `internal/k8s/client.go` and the `internal/k8s/*.go` siblings show every outbound call (all of them target the in-cluster API server or Prometheus route).
- `internal/server/server.go` shows the route table; nothing forwards data outside.
- `internal/auth/session.go` shows the session store (in-memory map, expiry, cleanup).
- Search for `http.NewRequest`, `os.Create`, `os.WriteFile`, `os.OpenFile` in the tree; you will find no calls writing cluster data to disk.

Found something this document does not cover, or contradicts? Open an issue or follow the reporting flow in `SECURITY.md`.
