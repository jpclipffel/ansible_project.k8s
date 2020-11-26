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

## How to

### How do I deploy a cluster ?

> These instructions assumes that you have already onboarded your
  servers into Ansible.

* Onboard the servers into an Ansible inventory in the correct groups:
  * `k8s_masters`
  * `k8s_workers`
* Provide (at least) the minimal cluster configuration in the Ansible inventory
  variables file.

  Example inventory structure:

  ```
  ../inventories/your_k8s/        # Inventory directory
  |
  \__ hosts                       # List of hosts in their groups
  |
  \__ group_vars/                 # Inventory variables files directory
      |
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

## Playbooks

| Playbook                           | Description                                            | Ready for AWX |
|------------------------------------|--------------------------------------------------------|---------------|
| `playbooks/main.yml`               | Builds, scales and configures a Kubernetes cluster.    | Yes           |
| `playbooks/ceph.yml`               | Setup a Ceph `StorageClass` on a Kubernetes cluster.   | Yes           |
| `playbooks/cleanup.yml`            | Remove one or more nodes from a Kubernetes cluster.    | No            |
| `playbooks/selfservice/consul.yml` | Deploy Consul on a Kubernetes cluster.                 | No            |
| `playbooks/selservice/gitlab.yml`  | Add a Kubernetes cluster to a GitLab group or project. | Yes           |

---

### Playbook - `main.yml`

Builds, configures and scales a Kubernetes cluster.

The playbook starts by creating or scaling a cluster with the *masters* nodes.
Then, it joins the *workers* nodes to the cluster. Finally, it applies all
roles defined in variable `k8s_roles`.

You can avoid setting-up the cluster by running the playbook with only the
`apply` or `delete` tag (see tags specifications bellow).

#### Tags

| Tag        | Description                                                      |
|------------|------------------------------------------------------------------|
| `stats`    | Collect and set custom stats                                     |
| `setup`    | Setup (deploy, scale up, scale down) a Kubernetes cluster        |
| `remove`   | Remove the Kubernetes cluster                                    |
| `apply`    | Include the roles specified in `k8s_roles` with the `apply` tag  |
| `delete`   | Include the roles specified in `k8s_roles` with the `delete` tag |

#### Roles

Roles included by default:
| Role          | Description           |
|---------------|-----------------------|
| `docker_base` | Docker runtime        |
| `k8s_base`    | Kubernetes deployment |
| `k8s_helm`    | Helm deployment       |

Optional roles, to be added in `k8s_roles`:

| Role                 | Description                    |
|----------------------|--------------------------------|
| `k8s_calico`         | Calico CNI                     |
| `k8s_csi-rbd`        | Ceph CSI                       |
| `k8s_volsnap`        | K8S volume snapshot controller |
| `k8s_metrics-server` | K8S metrics server             |
| `k8s_metallb`        | MetalLB load balancer          |
| `k8s_istio`          | Istion service mesh            |
| `k8s_prometheus`     | Prometheus metrics collector   |
| `k8s_argocd`         | ArgoCD server                  |

#### Variables

| Variable                     | Type     | Required | Default           | Description                            |
|------------------------------|----------|----------|-------------------|----------------------------------------|
| `k8s_control_plane_endpoint` | `string` | Yes      |                   | Cluster endpoint (URL)                 |
| `k8s_roles`                  | `list`   | No       | `[]` (empty list) | Optional roles to apply on the cluster |

---

### Playbook - `restart.yml`

Cleanly restart a Kubernetes cluster.

This playbook runs on each node **one by one**:

* Drain (*cordon*) the node if its a worker node
* Stop the `kubelet` service
* Restart the `docker` service
* Start the `kubelet` service
* Uncordon (*un-drain*) the node if its a worker node

#### Skip tags

One can exclude a node type by specifying it in `--skip-tags`:

| Tag      | Description              |
|----------|--------------------------|
| `master` | Exclude all master nodes |
| `worker` | Exclude all worker nodes |

---

### Playbook - `ceph.yml`

Setup a Ceph `StorageClass` on a Kubernetes cluster.

| Role              | Description                      |
|-------------------|----------------------------------|
| `ceph_client_k8s` | Manage a Ceph RBD `StorageClass` |

---

### Playbook - `cleanup.yml`

Remove nodes from cluster and purge them.

The nodes must either:

* Have their variable `k8s_node_type` set on `delete`
* Be a member of the group `k8s_deleted`

---

### Playbook - `selservice/gitlab.yml`

Add a Kubernetes cluster to a GitLab group or project.

| Role         | Description                                             |
|--------------|---------------------------------------------------------|
| `k8s_gitlab` | Add or remove a cluster to a GitLab instance or project |

---

### Playbook - `selfservice/consul.yml`

Deploy and maintains a Consul servers cluster (outside of Kubernetes) and clients (inside Kubernetes).

The servers and clients components are deployed according to this table:

| Host group | Component           | Notes                                                                                             |
|------------|---------------------|---------------------------------------------------------------------------------------------------|
| `masters`  | Server (out of K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |
| `workers`  | Client (within K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |

#### Tags

The role `consul_base` is **always** included and thus the Consul cluster facts are **always** gathered.
This means that one may invoke the playbook with `apply` withou having to specify the cluster's hosts, key, etc.

| Tag        | Description                         |
|------------|-------------------------------------|
| `setup`    | Setup the Consul server cluster     |
| `teardown` | Teardowns the Consul server cluster |
| `apply`    | Deploys the Consul K8S clients      |
| `delete`   | Removes the Consul K8S clients      |

#### Variables

| Variable                      | Type     | Required | Description                                                                    |
|-------------------------------|----------|----------|--------------------------------------------------------------------------------|
| `consul_base_cluster_hosts`   | `list`   | No       | List of Consul cluster hosts.<br>Can be automatically deduced by the playbook. |
| `consul_base_conf_datacenter` | `string` | No       | Consul datacenter name.<br>Can be automatically deduced by the playbook.       |
| `consul_base_conf_encrypt`    | `string` | No       | Consul gossip encryption key.<br>Can be automatically deduced by the playbook. |
