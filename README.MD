# Manual Deployment

Manually deploying flux to cluster eg. Kind Dev Cluster

## Prerequisites

- secrets.yaml - Secret for git repository flux will use.
- values.yaml - Helm values for flux/flux2-sync.

> Create a secrets.yaml file with the following:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: flux-system
  namespace: flux-system
type: Opaque
stringData:
  identity: # Private SSH Key that has access to Git Repository
  identity.pub: # Public SSH Key that has access to Git Repository
  known_hosts: # Repository's known host entry
```

> Create a values.yaml file with the following:
```yaml
gitRepository:
  spec:
    url: "ssh://git@github.com/cbx0n/flux" # use flux git repository 
    secretRef:
      name: "flux-system"
    interval: "1m0s"
    ref:
      branch: "dev" # use git branch to pull from
kustomization:
  spec:
    interval: "30s"
    path: "./cluster/dev" # use path to the environment
    prune: true
```

## Installation

1. Use the [kubectl](https://kubernetes.io/docs/tasks/tools/) tool to create flux-system namespace.

```bash
kubectl create ns flux-system
```

2. Then apply the secrets file:

```bash
kubectl apply -f secrets.yaml
```

3. Using [Helm](https://helm.sh/docs/intro/install/) add the [Flux](https://github.com/fluxcd-community/helm-charts/tree/main/charts/flux2) repo

```bash
helm repo add flux https://fluxcd-community.github.io/helm-charts
```

4. Next install the [flux2](https://github.com/fluxcd-community/helm-charts/pkgs/container/charts%2Fflux2) helm chart:

```bash
helm install flux flux/flux2 -n flux-system
```

5. Finally install the [flux2-sync](https://github.com/fluxcd-community/helm-charts/pkgs/container/charts%2Fflux2-sync) helm chart:

```bash
helm install flux-sync flux/flux2-sync -f values.yaml -n flux-system
```