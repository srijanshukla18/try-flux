# Flux Journey

## Setup - 2026-02-05

### Environment
- **Local K8s**: Colima with k3s v1.33.4
- **Flux version**: v2.7.5 (latest stable 2.7.x)

### Flux CLI Installation

Installed Flux 2.7.5 alongside existing CLI to avoid breaking other clusters:

```bash
# Existing CLI (unchanged)
flux --version  # v0.41.2

# New CLI for 2.7.x
curl -sL "https://github.com/fluxcd/flux2/releases/download/v2.7.5/flux_2.7.5_darwin_arm64.tar.gz" | tar -xz
mv flux /usr/local/bin/flux27

flux27 --version  # v2.7.5
```

### Cluster Setup

```bash
# Switch to local cluster
kubectl config use-context colima

# Pre-flight check
flux27 check --pre

# Install Flux controllers
flux27 install --version=v2.7.5
```

Controllers installed:
- source-controller v1.7.4
- kustomize-controller v1.7.3
- helm-controller v1.4.5
- notification-controller v1.7.5

### Repository Structure

```
try-flux/
├── base/                          # Shared configurations
│   ├── apps/
│   │   └── kustomization.yaml
│   └── infrastructure/
│       └── kustomization.yaml
└── clusters/
    └── colima/                    # Cluster-specific configs
        ├── kustomization.yaml     # Main entry point
        ├── apps.yaml              # Apps Kustomization CR
        ├── infrastructure.yaml    # Infra Kustomization CR
        ├── apps/
        │   └── kustomization.yaml
        ├── infrastructure/
        │   └── kustomization.yaml
        └── flux-system/
            ├── gotk-components.yaml  # Flux controllers manifest
            ├── gotk-sync.yaml        # GitRepository + root Kustomization
            └── kustomization.yaml
```

### Key Design Decisions

1. **Dependency ordering**: `apps` depends on `infrastructure` - ensures infra (ingress, cert-manager, etc.) is ready before apps deploy

2. **Separation of concerns**:
   - `base/` for reusable configurations across clusters
   - `clusters/<name>/` for cluster-specific overlays

3. **GitOps sync**: `gotk-sync.yaml` configures Flux to watch `main` branch and reconcile from `./clusters/colima`

---

## Next Steps

- [ ] Initialize git repo and push to GitHub
- [ ] Add deploy key for Flux to pull from repo
- [ ] Add a sample app (e.g., podinfo) to test the pipeline
- [ ] Add infrastructure components (ingress-nginx, etc.)
