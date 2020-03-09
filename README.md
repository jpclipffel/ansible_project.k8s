# Ansible - Projects - Kubernetes

## Usage

This project contains multiples playbooks and helpers tasks lists. Runnable playbooks are documented in the next sections.

## Inventory setup

This project support two methods to distinguises Kubernetes nodes types.

* Using host groups (see table bellow)
* Using host variable `k8s_node_type` (see table bellow)

|Group name|`k8s_node_type`|Description|
|----------|---------------|-----------|
|`masters` |`master`       |Kubernetes master node|
|`workers` |`worker`       |Kubernetes worker node|
|`deleted` |`delete`       |Node may can be cleaned-up to be removed from cluster or to be transitioned from/to master/worker (see playbook `cleanup.yml`)|

The helper tasks list `helper_nodetype.yml` will set the correct node type and host group.

## Common variables

Those variables applies to all playbooks.

|Variable|Type|Required|Description|
|--------|----|--------|-----------|
|`k8s_node_type`|`string`|-|Required if hosts are not located in the groups `masters` or `workers`.|

---

## Playbook `main.yml`

Deploy and maintains a Kubernetes cluster. It starts by creating or scaling a cluster with the *masters* node. Then, it joins the *workers* nodes to the cluster.

Beside Kubernetes, the playbook will also install *Helm* and the required Python packages to interface with Kubernetes.

This playbook essentialy wraps the following components:

* Role [`k8s_base`](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)
* Role [`k8s_mesh`](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_mesh)

### Tags

Multiple tags can be used at once (e.g. `setup`, `apply`, `calico`, `istio` to simultaneously setup the cluster and deploy a service mesh).

|Tag|Description|
|---|-----------|
|`stats`|Collect and set custom stats|
|`setup`|Setup (bluids, scales, maintains) a Kubernetes cluster|
|`teardown`|Teardown a Kubernetes cluster|
|`apply`|Deploys the K8S **services**|
|`delete`|Removes the K8S **services**|

### Variables

|Variable|Type|Required|Description|
|--------|----|--------|-----------|
|`k8s_control_plane_endpoint`|`string`|Yes|K8S control plane FQDN|

---

## Playbook `cleanup.yml`

Remove nodes from cluster and purge them.

The nodes must either:

* Have their variable `k8s_node_type` set on `delete`
* Be a member of the group `deleted`

---

## Playbook `consul.yml`

Deploy and maintains a Consul servers cluster (outside of Kubernetes) and clients (inside Kubernetes).

The servers and clients components are deployed according to this table:

|Host group|Component|Notes|
|----------|---------|-----|
|`masters`|Server (out of K8S)|See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable|
|`workers`|Client (within K8S)|See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable|

This playbook reuse the following components:

* Role [`consul_base`](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/consul_base)

### Tags

The role `consul_base` is **always** included and thus the Consul cluster facts are **always** gathered.
This means that one may invoke the playbook with `apply` withou having to specify the cluster's hosts, key, etc.

|Tag|Description|
|---|-----------|
|`setup`|Setup the Consul server cluster|
|`teardown`|Teardowns the Consul server cluster|
|`apply`|Deploys the Consul K8S clients|
|`delete`|Removes the Consul K8S clients|

### Variables

|Variable|Type|Required|Description|
|--------|----|--------|-----------|
|`consul_base_cluster_hosts`|`list`|No|List of Consul cluster hosts.<br>Can be automatically deduced by the playbook.|
|`consul_base_conf_datacenter`|`string`|No|Consul datacenter name.<br>Can be automatically deduced by the playbook.|
|`consul_base_conf_encrypt`|`string`|No|Consul gossip encryption key.<br>Can be automatically deduced by the playbook.|

---

## Playbook `manifest.yml`

Apply or delete a Kubernetes manifest.

### Tags

|Tag|Description|
|---|-----------|
|`apply`|Applies the provided `manifest`|
|`delete`|Deletes the given `manigest`|

### Variables

Please note that the playbook **does not asserts** the manifest variables.
If a variable used in the manifest file is missing, the playbook will fails.

|Variable|Type|Required|Description|
|--------|----|--------|-----------|
|`manifest`|`string`, file system path|Yes|Path to manifest template file (should be located under project's `templates/` directory).|

---

## Tasks list `helper_kubeconfig.yml`

Set the variable `k8s_kubeconfig`.

---

## Tasks list `helper_inventory.yml`

Ensure that the hosts are located into the correct groups and have the correct `k8s_node_type` variable value.

|Group name|Variable `k8s_node_type`|
|----------|------------------------|
|`masters` |`master`                |
|`workers` |`worker`                |
