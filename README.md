# fluxcd-eck-test

This repository contains the configuration files to deploy Elastic Cloud on Kubernetes (ECK) using FluxCD in the development environment (`kind` cluster). The setup leverages GitOps principles to automatically deploy and manage ECK components like Elasticsearch and Kibana using Kustomize overlays.

## Prerequisites

- **FluxCD v2** installed and bootstrapped on the `kind` cluster.
- **Kustomize** for managing environment-specific configurations.
- **Helm** for deploying the ECK stack.

## Repository Structure

```bash
.
├── README.md
├── base
│   └── eck                         # Base configuration shared between environments
│       ├── eck-operator.yaml       # HelmRelease for ECK Operator
│       ├── eck-stack.yaml          # HelmRelease for ECK stack (Elasticsearch, Kibana)
│       ├── kustomization.yaml      # Base Kustomization
│       ├── ns-elastic-stack.yaml   # elastic-stack namespace definition
│       ├── ns-elastic-system.yaml  # elastic-system namespace definition
│       └── source.yaml             # HelmRepository for ECK components
└── clusters
    ├── aks-prod                    # Testing, ignore for now
    │   │   ├── ...
    └── kind-cluster                # Dev environment (kind cluster)
        ├── apps.yaml               # dev-kustomization
        ├── flux-system
        │   ├── gotk-components.yaml
        │   ├── gotk-sync.yaml
        │   └── kustomization.yaml
        └── kustomization.yaml
```

## Base Configuration

The `base/eck/` directory contains shared configuration for deploying Elasticsearch and Kibana using Helm. The `eck-stack.yaml` defines the HelmRelease for deploying these components, and the base `kustomization.yaml` is referenced by the dev environment.

## Kustomize Overlay for Dev

The `clusters/kind-cluster` overlay is used for the local kind cluster. A few things to remember:
- Kibana is exposed using a `NodePort` service
- The disk size for Elasticsearch (`elasticsearch-data`) is set to 5Gi

## GitOps Deployment on Kind Cluster

#### 1. Fork this repository

   Instructions to do so can be found [here](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo)

   
#### 3. Kind cluster configuration

   Use the following configuration for your kind cluster

   ```yml
   kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker
   ```

#### 3. Start Kind cluster

   ```bash
   kind create cluster --name ksm-cluster-1-28 --config kind-cluster.yaml --image kindest/node:v1.28.13
   ```

#### 4. Install FluxCD and bootstrap it in your `ksm-cluster-1-28`

   [Create a GitHub Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic)

   ```bash
   export GITHUB_USER=<github-user>
   export GITHUB_TOKEN=<github-personal-access-token>
   export GITHUB_REPO=<your-forked-repo>
   flux bootstrap github --owner=${GITHUB_OWNER} --repository=${GITHUB_REPO} --branch=main --path=./clusters/dev --personal
   ```

## Monitoring

- **Check Kustomizations**

```bash
flux get kustomizations --watch
```

- **Check HelmReleases**

  ```bash
  flux get helmreleases -n elastic-system
  ```

- **Monitor Flux logs**

```bash
kubectl logs -f -n flux-system deploy/source-controller
```

## Contributing

Feel free to contribute to this repository by submitting pull requests or reporting issues

## License 

##### MIT
