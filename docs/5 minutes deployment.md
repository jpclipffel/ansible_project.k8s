# 5 minutes deployment

This document will guide you in the deployment of a production-ready Kubernetes clusters in less than 5 minutes.

It assumes you have a wrote or generated a K8S inventory, as explained in [the inventory documentation](Inventory.md).

## Create and configure the virtual machines

| Key              | Value                     |
|------------------|---------------------------|
| Ansible project  | `infrastructure`          |
| Ansible playbook | `playbooks/vm_vmware.yml` |
| Hosts group      | `vmware_guests`           |

```bash
ansible-playbook \
    # Select the inventory
    -i inventories/k8s_test/ \
    # Select the playbook
    projects/infra/playbooks/vm_vmware.yml \
    # Only target vrtual machines guests
    -l 'vmware_guests' \
    # Require to setup (create) and init (configure) the virtual machines
    --tags 'setup,init' \
    # vCenter username, password and VM username, password and new passwor to setup
    -e '{\
            "vmware_guest_username": "*****", \
            "vmware_guest_password": "*****", \
            "vmware_guest_vm_username":"*****", \
            "vmware_guest_vm_password": "*****", \
            "vmware_guest_init_password": "*****" \
        }'
```

## Setup the Ansible service account

> This step is optional if you already have configured the servers and onboarded them into Ansible / AWX CI/CD process.

| Key              | Value                |
|------------------|----------------------|
| Ansible project  | `infrastructure`     |
| Ansible playbook | `playbooks/main.yml` |
| Hosts group      | `k8s`                |

```bash
ansible-playbook \
    # Select the inventory
    -i inventories/k8s_test/ \
    # Select to playbook
    projects/infra/playbooks/main.yml \
    # Setup (configure) the hosts
    --tags 'setup' \
    # As this is a first connection, account and sudo password has to be provided manually.
    # CI/CD does NOT requires these two flags (-kK).
    -kK \
    # Setup the Ansible service account anem and public key (password authentication is disabled by default).
    # Replace 'ansible_user' and 'system_ansible_public_key_file' with the appropriate value.
    # Note that by default, the Ansible service account name is 'ansctl'.
    -e '{\
            "ansible_user": "ubuntu", \
            "system_ansible_public_key_file": "/home/ansctl/.ssh/ansctl.rsa.pub" \
        }'
```

## Create a Keepalived cluster

> This step is optional if you use an alternative way to provide HA to the K8S masters node & API  (e.g. HTTP load balancer like nginx).

| Key              | Value                      |
|------------------|----------------------------|
| Ansible project  | `infrastructure`           |
| Ansible playbook | `playbooks/keepalived.yml` |
| Hosts group      | `k8s`                      |

```bash
ansible-playbook \
    # Select the inventory
    -i inventories/k8s_test/ \
    # Select the playbook
    projects/infra/playbooks/keepalived.yml \
    # Only target the master nodes
    -l 'k8s_masters' \
    # Setup (install and configure) Keepalived
    --tags 'setup' \
    # Select the service account
    -e ansible_user=ansctl
```


## Create and scale a Kubernetes cluster

> This step is optional if you already have build a cluster.

| Key              | Value                |
|------------------|----------------------|
| Ansible project  | `kubernetes`         |
| Ansible playbook | `playbooks/main.yml` |
| Hosts group      | `k8s`                |

```bash
ansible-playbook \
    # Select the inventory
    -i inventories/k8s_test/ \
    # Select the playbook
    projects/k8s/playbooks/main.yml \
    # Build or scale the cluster
    --tags 'setup' \
    # Select the service account
    -e ansible_user=ansctl
```

## Configure a Kubernetes cluster

This step will configure a K8S cluster. You can (and should) run it on a schedule to ensure the cluster is kept up to date.

| Key              | Value                |
|------------------|----------------------|
| Ansible project  | `kubernetes`         |
| Ansible playbook | `playbooks/main.yml` |
| Hosts group      | `k8s`                |

```bash
ansible-playbook \
    # Select the inventory
    -i inventories/k8s_test/ \
    # Select the playbook
    projects/k8s/playbooks/main.yml \
    # Configure the cluster
    --tags 'apply' \
    # Select the service account
    -e ansible_user=ansctl
```
