# AWXLab - Project - Kubernetes

Deploy and maintains the AWXLab Kubernetes infrastructure.

## Playbooks

|Playbook|Description|Roles|
|--------|-----------|-----|
|`main.yml`|Deploy and maintain the K8S infrastructure|[Docker](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/docker_base)<br>[K8S](https://git.dt.ept.lu/jpclipffel/awxlab-roles-common/tree/master/k8s_base)|


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
