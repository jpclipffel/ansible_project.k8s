# Playbook - `restart.yml`

Properly restart a Kubernetes cluster.

This playbook can restarts a whole cluster (`--tags all`), the master nodes
(`--tags masters`) or the worker nodes (`--tags workers`).

The nodes are **always** restarted one by one (Ansible's `serial` set to `1`),
starting with the master nodes and ending with the worker nodes.

Each node undergoes the following operations:

* Node is drained
* `kubelet` deamon is reloaded amd restarted
* If the node is a master node, Ansible waits 120 seconds max for the API
  server to respond
* The node is uncordoned
* Ansible waits for 60 seconds to let the K8S services restart

## Tags

One can exclude a node type by specifying it in `--skip-tags`:

| Tag       | Description          |
|-----------|----------------------|
| `all`     | Restart all nodes    |
| `masters` | Restart master nodes |
| `workers` | Restart worker nodes |
