# Inventory

Kubernetes clusters are mapped in Ansible inventories. The typical inventory structure is:

```
../inventories/my_k8s_cluster/      # Inventory directory
\__ hosts                           # Cluster topology: masters & workers gropups and hosts, IP, etc.
\__ group_vars/
    \__ k8s.yml                     # K8S cluster & cluster components variables
    \__ vmware_guests.yml           # Virtual machines variables
```

##  Cluster topology - `hosts`

This file contains the list of cluster's hosts, organized in **hosts groups**:

| Group name        | Required | Description                                          | Group vars          |
|-------------------|----------|------------------------------------------------------|---------------------|
| `vmware_guests`   | No       | VMWare virtual machines                              | `vmware_guests.yml` |
| `k8s`             | Yes      | Root group for all K8S nodes (master, workers, etc.) | `k8s.yml`           |
| `k8s.k8s_masters` | Yes      | K8S masters nodes                                    | `k8s.yml`           |
| `k8s.k8s_workers` | Yes      | K8S workers nodes                                    | `k8s.yml`           |

Example:

```yaml
all:
  children:
    # Virtual machines
    # Reference: group_vars/vmware_guests.yml
    vmware_guests:
      hosts:
        k8s-test-master-01.dt.ept.lu:
          vmware_guest_name: "k8s-test-master-01"
          vmware_guest_init_netplan_ip: "172.29.252.196/24"
        k8s-test-master-02.dt.ept.lu:
          vmware_guest_name: "k8s-test-master-02"
          vmware_guest_init_netplan_ip: "172.29.252.197/24"
        k8s-test-master-03.dt.ept.lu:
          vmware_guest_name: "k8s-test-master-03"
          vmware_guest_init_netplan_ip: "172.29.252.188/24"
        k8s-test-worker-01.dt.ept.lu:
          vmware_guest_name: "k8s-test-worker-01"
          vmware_guest_init_netplan_ip: "172.29.252.198/24"
        k8s-test-worker-02.dt.ept.lu:
          vmware_guest_name: "k8s-test-worker-02"
          vmware_guest_init_netplan_ip: "172.29.252.199/24"
    # Kubernetes cluster
    # Reference: group_vars/k8s.yml
    k8s:
      children:
        k8s_masters:
          hosts:
            k8s-test-master-01.dt.ept.lu:
            k8s-test-master-02.dt.ept.lu:
            k8s-test-master-03.dt.ept.lu:
        k8s_workers:
          hosts:
            k8s-test-worker-01.dt.ept.lu:
            k8s-test-worker-02.dt.ept.lu:
```

## VMWare virtual machines - `group_vars/vmware_guests.yml`

> You can omit this section if you are not deploying on VMWare.

This file contains the common hosts configuration to create and configure the cluster's virtual machines.

The variables defined here are used nearly exclusively by the [`vmware_guest`](https://git.dt.ept.lu/ict-infra/iac/ansible/roles/vmware_guest) Ansible role. Please refer to [this documentation](https://git.dt.ept.lu/ict-infra/iac/ansible/roles/vmware_guest/-/blob/master/README.md) for more details.

## K8S clusters - `group_vars/k8s.yml`

This file contains the K8S cluster configuration and the various cluster components configuration.
