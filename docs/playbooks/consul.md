### Playbook - `selfservice/consul.yml`

Deploy and maintains a Consul servers cluster (outside of Kubernetes) and clients (inside Kubernetes).

The servers and clients components are deployed according to this table:

| Host group | Component           | Notes                                                                                             |
|------------|---------------------|---------------------------------------------------------------------------------------------------|
| `masters`  | Server (out of K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |
| `workers`  | Client (within K8S) | See section *Inventory setup* for more details regarding host groups and `k8s_node_type` variable |

#### Tags

The role `consul_base` is **always** included and thus the Consul cluster facts are **always** gathered.
This means that one may invoke the playbook with `apply` withou having to specify the cluster's hosts, key, etc.

| Tag        | Description                         |
|------------|-------------------------------------|
| `setup`    | Setup the Consul server cluster     |
| `teardown` | Teardowns the Consul server cluster |
| `apply`    | Deploys the Consul K8S clients      |
| `delete`   | Removes the Consul K8S clients      |

#### Variables

| Variable                      | Type     | Required | Description                                                                    |
|-------------------------------|----------|----------|--------------------------------------------------------------------------------|
| `consul_base_cluster_hosts`   | `list`   | No       | List of Consul cluster hosts.<br>Can be automatically deduced by the playbook. |
| `consul_base_conf_datacenter` | `string` | No       | Consul datacenter name.<br>Can be automatically deduced by the playbook.       |
| `consul_base_conf_encrypt`    | `string` | No       | Consul gossip encryption key.<br>Can be automatically deduced by the playbook. |
