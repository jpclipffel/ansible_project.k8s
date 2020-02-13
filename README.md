# AWXLab - Project - Kubernetes

Deploy and maintains the AWXLab Kubernetes infrastructure.

## Run

Tags can be associated when used with the same playbook.

|Playbook|Tags|Description|
|--------|----|-----------|
|`main.yml`|`install`, `configure`|Setup the Kubernetes nodes (hosts)|
|`main.yml`|`bootstrap_master`|Create a Kubernetes cluster|
|`main.yml`|`bootstrap_worker`|Add *workers* nodes to a Kubernetes cluster|

## Variables

|Variable|Type|Required|Description|
|--------|----|--------|-----------|
|`facted.kubernetes.role`|`string`|Yes|Kubernetes node type (`master` or `worker`)|
|`control_plane_endpoint`|`string`|Yes|Control plane FQDN|
|`kubeconfig`|`string`|No|Path to the node's configuration file<br>Defaults to Ansible user's `$HOME/.kube/admin.conf`|

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
