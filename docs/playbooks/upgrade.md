# Playbook - `upgrade/`

Upgrades a Kubernetes cluster.

The playbook `upgrade/main.yml` and the associated tasks (`upgrade/*.yml`)
upgrades a Kubernetes cluster to a chosen version.

The process is not fully automated as some checks **should really** be
performed by a Kubernetes administrator.

## Usage

### Initialize

To do.

### Primary master node

To do.

### Master nodes

To do.

### Worker nodes

To do.

## Tags

| Tag       | Description                                        |
|-----------|----------------------------------------------------|
| None      | Run pre-flight checks                              |
| `primary` | Initializes the upgrade on the primary master node |
| `masters` | Upgrade all master nodes                           |
| `workers` | Upgrade all worker nodes                           |

## Variables

| Variable           | Type     | Required | Default | Description                                                     |
|--------------------|----------|----------|---------|-----------------------------------------------------------------|
| `k8s_next_version` | `string` | Yes      |         | Kubernetes version to upgrade to in `major.minor` (e.g. `1.20`) |

