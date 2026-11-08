# dr-demo

Two-site warm standby with ArgoCD: a failover is a Git commit.

One app (an HTTP echo that answers with the name of the site serving it) runs on two clusters:

| Cluster | Overlay | Replicas |
|---|---|---|
| saturn-adbn1 | `overlays/primary` | 1 |
| jupiter-wsdc1 | `overlays/standby` | 0 |

The standby site has everything in place (namespace, Service, Ingress) except a running pod.
A failover is one commit that swaps the two replica counts; failing back is a revert. ArgoCD
self-heal undoes a manual `kubectl scale`, so the only way to move the workload is through Git.

How traffic finds the active site (DNS, a global load balancer, a route) is outside this demo.

```
base/                  Deployment + Service + Ingress
overlays/primary/      replicas 1, answers "saturn-adbn1 (primary)"
overlays/standby/      replicas 0, answers "jupiter-wsdc1 (standby)"
argocd/project.yaml    AppProject: namespace dr-demo on the two clusters only
argocd/applicationset.yaml   one Application per cluster, each with its overlay
```

## Install

On the ArgoCD cluster:

```bash
kubectl apply -f argocd/project.yaml -f argocd/applicationset.yaml
```

## Teardown

```bash
kubectl -n argocd delete applicationset dr-demo-failover
kubectl -n argocd delete appproject dr-demo
# CreateNamespace does not delete the namespace with the app: on both clusters
kubectl delete namespace dr-demo
```
