# Playbook - `ceph.yml`

Setup a Ceph `StorageClass` on a Kubernetes cluster.

This playbook wraps the role `ceph_client_k8s` in two steps:

* Gather the Ceph cluster information
* Build and install a RBD `StorageClass` from the gathered information

This playbook uses two groups from the provided inventory(ies):

* A Ceph cluster / *mons* nodes with the group `ceph_mons`
* A Kubernetes cluster / master nodes with the group `k8s_masters`

The Ceph cluster information are automatically gathered by the role
`ceph_client_k8s` (from the tasks file `ceph_facts.yml`), **if and only if**
at least one host is given in the `ceph_mons` group.

You have three options when calling the playbook to setup the `StorageClass`:

* Provide two inventories to Ansible when calling the playbook (one for K8S
  and another for Ceph:
  `ansible-playbook -i <../inventories/k8s> -i <../inventories/ceph> playbooks/ceph.yml --tags [...]`
* Provide the Ceph cluster facts as variables.
  See the role `ceph_client_k8s` documentation for the required variables.
* Add the Ceph cluster nodes in you K8S inventory.
  This is **not a good solution** for Ceph clusters hosted out of K8S.

## Tags

| Tag      | Description                                     |
|----------|-------------------------------------------------|
| `apply`  | Builds then deploy or update the `StorageClass` |
| `delete` | Remove the `StorageClass`                       |

## Roles

| Role              | Description                      |
|-------------------|----------------------------------|
| `ceph_client_k8s` | Manage a Ceph RBD `StorageClass` |
