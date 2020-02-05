# AWXLab - Project - Kubernetes

Deploy and maintains the AWXLab Kubernetes infrastructure.

## Run

|Playbook|Tags|Description|Roles|
|--------|----|-----------|-----|
|`main`|`install`|Install Docker and K8S on hosts|[docker_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/docker_base)<br>[k8s_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)|
|`main`|`configure`|Configure hosts to run Docker and K8S|[docker_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/docker_base)<br>[k8s_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)|
|`main`|`bootstrap_master`|Bootstrap the master(s) node(s) (with `k8s_base_node_type` set on `master`)<br>**Warning**: Node(s) will be reseted|[k8s_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)|
|`main`|`bootstrap_worker`|Bootstrap the worker(s) node(s) (with `k8s_base_node_type` set on `worker`)<br>**Warning**: Node(s) will be reseted|[k8s_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)|
|`main`|`apply`|Apply (deploy) the AWXLab K8S infrastructure<br>**Resources**: Calico, Istio, KNative|-|
|`main`|`delete`|Delete the AWXLab K8S infrastructure<br>**Resources**: Calico, Istio, KNative|-|

## Variables

|Variable|Type|Required|Description|
|--------|----|--------|-----------|
|`facted.kubernetes.role`|`string`|Yes|Kubernetes node type (`master` or `worker`)|

## Usage

### Ansible Tower

* Create a new **project** using **Git** as a SCM
* Provide the following configuration:

|Key|Value|
|---|-----|
|SCM URL|`https://git.dt.ept.lu/jpclipffel/awxlab-project-k8s.git`|
|SCM Credential|Your SCM credentials (see the *Hints* section)|
|SCM update options|Check *Update revison on launch*|

---

## Hints

### Create a GitLab credential for Ansible Tower

#### 1 - On GitLab

* Create a new access token (go to *Settings*, *Access Tokens*)
* Provide at the least the `read_repository` scope
* Save the token attributes in a safe location (e.g. *KeePass*)

#### 2 - On Ansible Tower

* Create a new credential of type **Source Control**
* Provide the following configuration:

|Key|Value|
|---|-----|
|Username|Your token name|
|Password|Your token value|
