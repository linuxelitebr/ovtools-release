# OVTools

**OpenShift Virtualization Tools**: 
A web-based inventory tool for OpenShift Virtualization environments, inspired by RVTools.

![graphs](docs/graphs.png)

![ovtools](docs/screenshot.png)

## Features

- **VM Inventory**: List all VMs with status, resources, IPs, and guest agent info
- **Node Overview**: Cluster nodes with capacity, workload distribution, and health
- **Snapshots**: Track VM snapshots with age warnings
- **Health Checks**: Identify configuration issues and resource problems
- **Export**: Download data as Excel (XLSX) or CSV
- **Auto-refresh**: 30-second automatic refresh without changing active tab
- **Multi-user**: Session-based authentication using user's own credentials
- **Etc**

## Authentication

OVTools supports two authentication methods:

| Method | Description | Use Case |
|--------|-------------|----------|
| **Token** | Bearer token from `oc whoami -t` | Quick access, CLI users |
| **Kubeconfig** | Paste kubeconfig content | Full config with context |

**Important**: Your credentials are used to connect to the cluster but are NOT stored. Sessions expire after 1 hour.

### RBAC

OVTools uses **delegated authentication** - each user sees only what their RBAC permissions allow:
- Cluster-admin sees all namespaces
- Namespace-scoped users see only their namespaces
- No special service account permissions required

## Quick Start

Run locally:

```sh
podman run -d --name ovtools-app -p 8080:8080  quay.io/andrerocha_redhat/ovtools:latest
```

```sh
podman ps
# CONTAINER ID  IMAGE                                     COMMAND               CREATED         STATUS         PORTS                   NAMES
# 1814c4cdd769  quay.io/andrerocha_redhat/ovtools:latest  -bind 0.0.0.0 -po...  17 seconds ago  Up 17 seconds  0.0.0.0:8080->8080/tcp  ovtools-app
```

Dev Mode (Demo Mode):

```sh
podman run --env OVTOOLS_DEV_MODE=true -d --name ovtools-app -p 8080:8080  quay.io/andrerocha_redhat/ovtools:latest
```

#### Access

```sh
http://<IP>:8080
```

### OpenShift Deployment

Create namespace:

```sh
oc new-project ovtools
```

Deploy:

```sh
oc apply -f deploy/openshift/
```

Get route URL:

```sh
oc get route ovtools -o jsonpath='{.spec.host}'
```

## Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `-bind` | `0.0.0.0` | Listen address |
| `-port` | `8080` | HTTP port |
| `-cache-ttl` | `60` | Cache TTL in seconds |
| `-version` | - | Show version and exit |

## Architecture

![architecture](docs/diagram.png)

## Data Collected

### VMs Tab
- Name, Namespace, Status
- vCPUs, Memory, Disks
- Node placement, Primary IP
- Guest OS (from guest agent)
- Guest Agent status and version
- Etc

### Nodes Tab
- Name, Status, Role
- CPU/Memory capacity and allocatable
- VM count, vCPU/vRAM totals
- Overcommit ratios
- Taints and schedulability
- Etc

### Snapshots Tab
- Name, VM association
- Creation date, Age
- Ready status, Size

### Health Tab
- VM configuration issues
- Resource warnings
- Snapshot age alerts
- Node problems

## License

Apache 2.0

## Author

Andre Rocha ⚡️ Forged in Chaos
