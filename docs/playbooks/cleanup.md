# Playbook - `cleanup.yml`

Removes nodes from a cluster and purge them.

The nodes to remove must either:

* Have their variable `k8s_node_type` set on `delete`
* Be a member of the group `k8s_deleted`
