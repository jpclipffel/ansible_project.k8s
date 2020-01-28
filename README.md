# AWXLab - Project - Kubernetes

Deploy and maintains the AWXLab Kubernetes infrastructure.

## Playbooks

|Playbook|Description|Roles|
|--------|-----------|-----|
|`install.yml`|Deploy and maintain the K8S infrastructure|[docker_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/docker_base)<br>[k8s_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)|
|`k8s_deploy_role`|Deploy a Kubernetes definition through an Ansible *role*|Common roles referenced as `common/<role_name>`, local as `<role_name>`|
|`reset`|Reset the Kubernetes infrastructure|[k8s_base](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)|

## Usage

### Ansible Tower

* Create a new project using **Git** as a SCM
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
