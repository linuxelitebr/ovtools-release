# Security Policy

## Reporting a Vulnerability

If you find a security issue in ovtools, please report it privately.

**Do NOT open a public GitHub issue for security vulnerabilities.**

Use GitHub's private vulnerability reporting on the repository (Security tab > Report a vulnerability), or email the maintainer at `andre.rocha@linuxelite.com.br`.

Please include enough detail to reproduce the issue: ovtools version, deployment mode (in-cluster, local binary), and the cluster setup (OpenShift version, KubeVirt version if relevant). A proof-of-concept manifest or curl invocation helps a lot.

Expect an acknowledgement within a few business days. A fix lands as soon as the impact and scope are understood; high-severity issues get a patch release outside the regular cadence.

## Scope

ovtools is a server-side dashboard that calls the Kubernetes API on the user's behalf. The security surface is roughly:

- The HTTP server bound to `0.0.0.0:8080` (or `127.0.0.1:8080` when running behind the oauth-proxy sidecar).
- The `/api/*` routes, all wrapped by `requireAuth` middleware.
- The Kubernetes client created per session, authenticated either by user-supplied token, kubeconfig, or oauth-proxy impersonation.
- The Prometheus client used for metrics, optionally pointed at a custom URL.

Out of scope:

- Vulnerabilities in OpenShift, KubeVirt, or any upstream Kubernetes component (report those to the respective upstream).
- Issues that require local shell access to the machine running ovtools, or cluster-admin already present on the target cluster.
- Bundled third-party libraries with known CVEs that have not landed in a release yet. Dependabot opens those PRs automatically; see `THIRD_PARTY_LICENSES.md` for the inventory.

## Supported Versions

Only the latest minor line gets security fixes. Older lines are best-effort, no commitment.

| Version | Supported |
|---------|-----------|
| 0.3.x   | Yes       |
| 0.2.x   | No        |
| 0.1.x   | No        |

## Best Practices for Operators

- Deploy ovtools in-cluster with `deploy/openshift/deployment.yaml`. The oauth-proxy sidecar enforces login through the OpenShift OAuth server and Kubernetes RBAC. Running the binary directly on a workstation is fine for one-off use but skips that protection.
- Do not expose the ovtools port to the public internet. If you must, put it behind an authenticated reverse proxy with TLS.
- ovtools never stores tokens or kubeconfigs on disk. Sessions live in memory and expire after 1 hour. If the process restarts, everyone logs in again.
- TLS verification is on by default. The `-insecure` option exists for lab clusters with self-signed certs; do not use it in production.
- Treat the dashboard URL like any other Kubernetes management plane: the user behind it has the RBAC of their own OpenShift account, nothing more, nothing less.
