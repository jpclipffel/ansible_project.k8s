# Ansible project - Kubernetes

Ansible project to build and manage Kubernetes clusters.

## Inventory setup

This project support two methods to distinguises Kubernetes nodes types.

* Using host groups (see table bellow)
* Using host variable `k8s_node_type` (see table bellow)

| Group name    | `k8s_node_type` | Description                             |
|---------------|-----------------|-----------------------------------------|
| `k8s_masters` | `master`        | Kubernetes master node                  |
| `k8s_workers` | `worker`        | Kubernetes worker node                  |
| `k8s_deleted` | `delete`        | Nodes marked to be removed from cluster |

The helper tasks list `helper_inventory.yml` will set the correct node type and host group.

---

## Playbooks

| Playbook       | Description                                            | Ready for AWX |
|----------------|--------------------------------------------------------|---------------|
| `main.yml`     | Builds, scales and configures a Kubernetes cluster.    | **Yes**       |
| `gitlab.yml`   | Add a Kubernetes cluster to a GitLab group or project. | **Yes**       |
| `ceph.yml`     | Setup a Ceph `StorageClass` on a Kubernetes cluster.   | **Yes**       |
|                |                                                        |               |
| `cleanup.yml`  |                                                        | No            |
| `consul.yml`   |                                                        | No            |
| `manifest.yml` |                                                        | No            |

### Playbook - `main.yml`

Builds, scales and configures a Kubernetes cluster.

The playbook starts by creating or scaling a cluster with the *masters* nodes. Then, it joins the *workers* nodes to the cluster. Finally, it applies all roles defined in variable `k8s_roles`.

#### Tags

Multiple tags can be used at once (e.g. `setup`, `apply`, `calico`, `istio` to simultaneously setup the cluster and deploy a service mesh).

| Tag        | Description                                                      |
|------------|------------------------------------------------------------------|
| `stats`    | Collect and set custom stats                                     |
| `setup`    | Setup (deploy, scale up, scale down) a Kubernetes cluster        |
| `remove`   | Remove the Kubernetes cluster                                    |
| `apply`    | Include the roles specified in `k8s_roles` with the `apply` tag  |
| `delete`   | Include the roles specified in `k8s_roles` with the `delete` tag |

#### Roles

Roles included by default:
| Role          | Documentation                                                 |
|---------------|---------------------------------------------------------------|
| `docker_base` | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/docker_base |
| `k8s_base`    | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/k8s_base    |
| `k8s_helm`    | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/k8s_helm    |

Optional roles, included with `k8s_roles`:

| Role             | Documentation                                                    |
|------------------|------------------------------------------------------------------|
| `k8s_calico`     | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/k8s_calico     |
| `k8s_istio`      | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/k8s_istio      |
| `k8s_prometheus` | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/k8s_prometheus |

#### Variables

| Variable                     | Type     | Required | Default | Description                            |
|------------------------------|----------|----------|---------|----------------------------------------|
| `k8s_control_plane_endpoint` | `string` | Yes      |         | Cluster endpoint (URL)                 |
| `k8s_roles`                  | `list`   | No       |         | Optional roles to apply on the cluster |

---

### Playbook - `gitlab.yml`

Add a Kubernetes cluster to a GitLab group or project.

| Role         | Documentation                                                |
|--------------|--------------------------------------------------------------|
| `k8s_gitlab` | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/k8s_gitlab |

---

### Playbook - `ceph.yml`

Setup a Ceph `StorageClass` on a Kubernetes cluster.

| Role              | Documentation                                                     |
|-------------------|-------------------------------------------------------------------|
| `ceph_client_k8s` | https://git.dt.ept.lu/ict-infra/iac/ansible/roles/ceph_client_k8s |

---

### Playbook - `cleanup.yml`

Remove nodes from cluster and purge them.

The nodes must either:

* Have their variable `k8s_node_type` set on `delete`
* Be a member of the group `k8s_deleted`

---

### Playbook - `consul.yml`

Deploy and maintains a Consul servers cluster (outside of Kubernetes) and clients (inside Kubernetes).

The servers and clients components are deployed according to this table:

| Host group | Component           | Notes                                                                                             |
|------------|---------------------|---------------------------------------------------------------------------------------------------|
| `masters`  | Server (out of K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |
| `workers`  | Client (within K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |

This playbook reuse the following components:

* Role [`consul_base`](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/consul_base)

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

---

### Playbook - `manifest.yml`

Apply or delete a Kubernetes manifest.

#### Tags

| Tag      | Description                     |
|----------|---------------------------------|
| `apply`  | Applies the provided `manifest` |
| `delete` | Deletes the given `manigest`    |

#### Variables

Please note that the playbook **does not asserts** the manifest variables.
If a variable used in the manifest file is missing, the playbook will fails.

| Variable   | Type                       | Required | Description                                                                                |
|------------|----------------------------|----------|--------------------------------------------------------------------------------------------|
| `manifest` | `string`, file system path | Yes      | Path to manifest template file (should be located under project's `templates/` directory). |

---

### Helper - `helper_kubeconfig.yml`

Set the variable `k8s_kubeconfig`.

---

## Helper - `helper_inventory.yml`

Ensure that the hosts are located into the correct groups and have the correct `k8s_node_type` variable value.

| Group name | Variable `k8s_node_type` |
|------------|--------------------------|
| `masters`  | `master`                 |
| `workers`  | `worker`                 |
