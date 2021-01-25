# Ansible project - Kubernetes

Ansible project to build and manage Kubernetes clusters.

## General concept

This project is a Kubernetes infrastructure-as-code implementation. From this
project standpoint, a K8S cluster is divided into 3 layers:

* **Foundation**: Docker, K8S binaries (e.g. `kubelet`), Helm, ...
* **Core**: The K8S cluster and its **base components**
* **Ecosystem**: The various resources (deployment, CRDs, ...) deployed during
  the cluster lifetime.

The various Ansible roles used by this project are either used for the cluster
foundation or core:

* Foundation: `k8s_docker`, `k8s_base`
* Core: `k8s_metallb`, `k8s_istio`, etc.

Whats happening at the ecosystem level is out of scope of this project.

---

## Playbooks

This project provides several playbooks.

| Playbook                           | Description                                            | Ready for AWX | Documentation                     |
|------------------------------------|--------------------------------------------------------|---------------|-----------------------------------|
| `playbooks/main.yml`               | Builds, scales and configures a Kubernetes cluster.    | Yes           | [Link](docs/playbooks/main.md)    |
| `playbooks/ceph.yml`               | Setup a Ceph `StorageClass` on a Kubernetes cluster.   | Yes           | [Link](docs/playbooks/ceph.md)    |
| `playbooks/gitlab.yml`             | Add a Kubernetes cluster to a GitLab group or project. | Yes           | [Link](docs/playbooks/gitlab.md)  |
| `playbooks/restart.yml`            | Restart a Kubernetes cluster.                          | Yes           | [Link](docs/playbooks/restart.md) |
| `playbooks/cleanup.yml`            | Remove one or more nodes from a Kubernetes cluster.    | No            | [Link](docs/playbooks/cleanup.md) |
| `playbooks/upgrade/`               | Upgrade a Kubernetes cluster.                          | No            | [Link](docs/playbooks/upgrade.md) |

---

## Inventory setup

This project support two methods to distinguish the Kubernetes nodes types:

* Using the hosts groups (see table bellow)
* Using the hosts variables `k8s_node_type` (see table bellow)

| Group name    | `k8s_node_type` | Description                             |
|---------------|-----------------|-----------------------------------------|
| `k8s_masters` | `master`        | Kubernetes master node                  |
| `k8s_workers` | `worker`        | Kubernetes worker node                  |
| `k8s_deleted` | `delete`        | Nodes marked to be removed from cluster |

The helper playbook `playbooks/helpers/inventory.yml` set the correct node type
and host group.

---

## How to

### How do I (re)deploy a cluster ?

> These instructions assumes that you have already onboarded your
  servers into Ansible.

> Take a look at the [5 minutes deployment guide](docs/5-minutes-deployment.md)

> Take a look at the [inventory guide](docs/inventory.md)

* Onboard the servers into an Ansible inventory in the correct groups:
  * `k8s_masters`
  * `k8s_workers`
* Provide (at least) the minimal cluster configuration in the Ansible inventory
  variables file.

  Example inventory structure:

  ```
  ../inventories/your_k8s/        # Inventory directory
  \__ hosts                       # List of hosts in their groups
  \__ group_vars/                 # Inventory variables files directory
      \__ k8s.yml                 # Kubernetes cluster configuration
  ```

  Example `hosts` contents:

  ```yaml
  all:
    children:
      # Kubernetes cluster
      # Reference: group_vars/k8s.yml
      k8s:
        children:
          k8s_masters:
            hosts:
              k8s-master-01.tld:
              k8s-master-02.tld:
              k8s-master-03.tld:
          k8s_workers:
            hosts:
              k8s-worker-01.tld:
              k8s-worker-02.tld:
              k8s-worker-03.tld:
              k8s-worker-04.tld:
  ```

  Example of a minimal `group_vars/k8s.yml` contents:

  ```yaml
  # This must resolve to one of the master node
  k8s_control_plane_endpoint: "k8s-master.tld"
  # Deploy at least a CNI on the cluster (e.g. Calico)
  k8s_roles:
    - k8s_calico
  ```

* Run the playbook `main.yml` with the `setup` and `apply` tags:

  ```shell
  ansible-playbook -i <inventory> playbooks/main.yml --tags 'setup,apply'
  ```

### How do I add a node (master or worker) ?

* Add the node(s) in the inventory groups `k8s_masters` or `k8s_workers`
* Run the playbook `main.yml` with the tag `setup`:

  ```shell
  ansible-playbook -i <inventory> playbooks/main.yml --tags 'setup'
  ```

### How do I remove a node (master or worker) ?

* Move the node(s) to remove to the group `k8s_deleted` in the inventory
* Run the playbook `cleanup.yml`:

  ```shell
  ansible-playbook -i <inventory> playbooks/cleanup.yml
  ```

---



---

