# Playbook - `upgrade/`

Upgrades a Kubernetes cluster.

The playbook `upgrade/main.yml` and the associated tasks (`upgrade/*.yml`)
upgrades a Kubernetes cluster to a chosen version.

The process is not fully automated as some checks **should really** be
performed by a Kubernetes administrator.

This playbook has been tested for the following versions:

* `1.17` to `1.18`
* `1.18` to `1.19`
* `1.19` to `1.20`

## Usage

This playbook may be called three consecuive times with different tags but with
the same `k8s_next_version` variable.

### Primary

This step *initializes* the upgrade on the *primary master node*.
This primary node is chosen among the cluster master nodes (first master node
in inventory to respond).

Example:

```shell
ansible-playbook -i <inventory> playbooks/upgrade/main.yml -e k8s_next_version='1.20' --tags 'primary'
```

The playbook will give you instructions to finishes the initialization on the
primary master node; It should instructs you to:

* Run `kubeadm upgrade plan` to validate that the upgrade process can be done
* Run `kubeadm upgrade apply <version>` to upgrade the manifests on the node

### Master nodes

This step upgrades all the master nodes **one by one**.

Example:

```shell
ansible-playbook -i <inventory> playbooks/upgrade/main.yml -e k8s_next_version='1.20' --tags 'masters'
```

### Worker nodes

This step upgrades all the workers nodes **one by one**.

Example:

```shell
ansible-playbook -i <inventory> playbooks/upgrade/main.yml -e k8s_next_version='1.20' --tags 'workers'
```

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
