# Playbook - `main.yml`

Builds, configures and scales a Kubernetes cluster.

The playbook starts by creating or scaling a cluster with the *masters* nodes.
Then, it joins the *workers* nodes to the cluster. Finally, it applies all
roles defined in variable `k8s_roles`.

You can avoid setting-up the cluster by running the playbook with only the
`apply` or `delete` tag (see tags specifications bellow).

## Tags

| Tag        | Description                                                      |
|------------|------------------------------------------------------------------|
| `stats`    | Collect and set custom stats                                     |
| `setup`    | Setup (deploy, scale up, scale down) a Kubernetes cluster        |
| `remove`   | Remove the Kubernetes cluster                                    |
| `apply`    | Include the roles specified in `k8s_roles` with the `apply` tag  |
| `delete`   | Include the roles specified in `k8s_roles` with the `delete` tag |

## Roles

Roles included by default:
| Role          | Description           |
|---------------|-----------------------|
| `docker_base` | Docker runtime        |
| `k8s_base`    | Kubernetes deployment |
| `k8s_helm`    | Helm deployment       |

Optional roles, to be added in `k8s_roles`:

| Role                 | Description                    |
|----------------------|--------------------------------|
| `k8s_calico`         | Calico CNI                     |
| `k8s_csi-rbd`        | Ceph CSI                       |
| `k8s_volsnap`        | K8S volume snapshot controller |
| `k8s_metrics-server` | K8S metrics server             |
| `k8s_metallb`        | MetalLB load balancer          |
| `k8s_istio`          | Istion service mesh            |
| `k8s_prometheus`     | Prometheus metrics collector   |
| `k8s_argocd`         | ArgoCD server                  |

## Variables

| Variable                     | Type     | Required | Default           | Description                            |
|------------------------------|----------|----------|-------------------|----------------------------------------|
| `k8s_control_plane_endpoint` | `string` | Yes      |                   | Cluster endpoint (URL)                 |
| `k8s_roles`                  | `list`   | No       | `[]` (empty list) | Optional roles to apply on the cluster |
