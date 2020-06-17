# Ansible - Projects - Kubernetes

## Usage

This project contains multiples playbooks and helpers tasks lists. Runnable playbooks are documented in the next sections.

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

## Common variables

Those variables applies to all playbooks.

| Variable        | Type     | Required | Description                                                     |
|-----------------|----------|----------|-----------------------------------------------------------------|
| `k8s_node_type` | `string` | No       | Required for nodes not in groups `k8s_masters` or `k8s_workers` |

---

## Playbook `main.yml`

Deplosy and maintains a Kubernetes cluster. It starts by creating or scaling a cluster with the *masters* node. Then, it joins the *workers* nodes to the cluster. Finally, it will apply all roles defined in variable `k8s_roles`.

### Tags

Multiple tags can be used at once (e.g. `setup`, `apply`, `calico`, `istio` to simultaneously setup the cluster and deploy a service mesh).

| Tag        | Description                                                      |
|------------|------------------------------------------------------------------|
| `stats`    | Collect and set custom stats                                     |
| `setup`    | Setup (deploy, scale up, scale down) a Kubernetes cluster        |
| `remove`   | Remove the Kubernetes cluster                                    |
| `apply`    | Include the roles specified in `k8s_roles` with the `apply` tag  |
| `delete`   | Include the roles specified in `k8s_roles` with the `delete` tag |

### Variables

| Variable                     | Type     | Required | Description            |
|------------------------------|----------|----------|------------------------|
| `k8s_control_plane_endpoint` | `string` | Yes      | K8S control plane FQDN |

---

## Playbook `cleanup.yml`

Remove nodes from cluster and purge them.

The nodes must either:

* Have their variable `k8s_node_type` set on `delete`
* Be a member of the group `k8s_deleted`

---

## Playbook `consul.yml`

Deploy and maintains a Consul servers cluster (outside of Kubernetes) and clients (inside Kubernetes).

The servers and clients components are deployed according to this table:

| Host group | Component           | Notes                                                                                             |
|------------|---------------------|---------------------------------------------------------------------------------------------------|
| `masters`  | Server (out of K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |
| `workers`  | Client (within K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |

This playbook reuse the following components:

* Role [`consul_base`](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/consul_base)

### Tags

The role `consul_base` is **always** included and thus the Consul cluster facts are **always** gathered.
This means that one may invoke the playbook with `apply` withou having to specify the cluster's hosts, key, etc.

| Tag        | Description                         |
|------------|-------------------------------------|
| `setup`    | Setup the Consul server cluster     |
| `teardown` | Teardowns the Consul server cluster |
| `apply`    | Deploys the Consul K8S clients      |
| `delete`   | Removes the Consul K8S clients      |

### Variables

| Variable                      | Type     | Required | Description                                                                    |
|-------------------------------|----------|----------|--------------------------------------------------------------------------------|
| `consul_base_cluster_hosts`   | `list`   | No       | List of Consul cluster hosts.<br>Can be automatically deduced by the playbook. |
| `consul_base_conf_datacenter` | `string` | No       | Consul datacenter name.<br>Can be automatically deduced by the playbook.       |
| `consul_base_conf_encrypt`    | `string` | No       | Consul gossip encryption key.<br>Can be automatically deduced by the playbook. |

---

## Playbook `manifest.yml`

Apply or delete a Kubernetes manifest.

### Tags

| Tag      | Description                     |
|----------|---------------------------------|
| `apply`  | Applies the provided `manifest` |
| `delete` | Deletes the given `manigest`    |

### Variables

Please note that the playbook **does not asserts** the manifest variables.
If a variable used in the manifest file is missing, the playbook will fails.

| Variable   | Type                       | Required | Description                                                                                |
|------------|----------------------------|----------|--------------------------------------------------------------------------------------------|
| `manifest` | `string`, file system path | Yes      | Path to manifest template file (should be located under project's `templates/` directory). |

--

## Playbook `gitlab.yml`

Integrates a Kubernetes cluster into a GitLab instance group or project.

---

## Tasks list `helper_kubeconfig.yml`

Set the variable `k8s_kubeconfig`.

---

## Tasks list `helper_inventory.yml`

Ensure that the hosts are located into the correct groups and have the correct `k8s_node_type` variable value.

| Group name | Variable `k8s_node_type` |
|------------|--------------------------|
| `masters`  | `master`                 |
| `workers`  | `worker`                 |
