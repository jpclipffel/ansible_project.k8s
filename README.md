# Ansible - Projects - Kubernetes

## Usage

This project contains multiples playbooks and helpers tasks lists. Runnable playbooks are documented in the next sections.

## Inventory setup

This project support two methods to distinguises between Kubernetes *masters* and *workers* nodes:

* The hosts may be located in a group named `masters` or `workers`
* The hosts may have the variable `k8s_node_type` set on `master` or `worker`

In both case, the helper tasks list `helper_nodetype.yml` will set the node type by (re)defining the variable `k8s_node_type`.

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
|`setup`|Setup (bluids, scales, maintains) a Kubernetes cluster|
|`teardown`|Teardown a Kubernetes cluster|
|`apply`|Deploys the K8S **services**|
|`delete`|Removes the K8S **services**|

### Variables

|Variable|Type|Required|Description|
|--------|----|--------|-----------|
|`k8s_control_plane_endpoint`|`string`|Yes|K8S control plane FQDN|

---

## Playbook `consul.yml`

Deploy and maintains a Consul servers cluster (outside of Kubernetes) and clients (within Kubernetes).

This playbook essentialy wraps the following components:

* Role [`consul_base`](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/consul_base)
* Playbook `helm.yml`

### Tags

|Tag|Description|
|---|-----------|
|`setup`|Setup the Consul server cluster|
|`teardown`|Teardowns the Consul server cluster|
|`apply`|Deploys the Consul K8S clients|
|`delete`|Removes the Consul K8S clients|
