# Playbook - `restart.yml`

Cleanly restart a Kubernetes cluster.

This playbook runs on each node **one by one**:

* Drain (*cordon*) the node if its a worker node
* Stop the `kubelet` service
* Restart the `docker` service
* Start the `kubelet` service
* Uncordon (*un-drain*) the node if its a worker node

## Ttags

One can exclude a node type by specifying it in `--skip-tags`:

| Tag      | Description              |
|----------|--------------------------|
| `master` | Exclude all master nodes |
| `worker` | Exclude all worker nodes |
