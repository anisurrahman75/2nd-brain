## GitOps Infrastructure & Applications Repository

This repository manages the `GitOps-based` deployment of both infrastructure components and application workloads across multiple environments (e.g., `staging`, `production`).

---

### Directory Structure

```bash
.
├── applications/                # Application HelmReleases (frontend, backend, postgres)
│   ├── base/                    # Base reusable manifests
│   └── overlays/                # Env-specific customizations (namespace, prefixes)
├── infrastructure/              # Infra HelmReleases & Operators
│   ├── base/                    # Base infra tools like CNPG, Velero, NGINX, CertManager
│   ├── overlays/                # Env-specific overlays with `namePrefix` and `namespace`
│   └── controllers/             # Optional raw YAML HelmReleases (non-Helm-managed)
├── clusters/                    # Cluster-level Flux wiring (gotk-sync + root Kustomizations)
│   └── <env>/                   # E.g., staging, production
│       ├── flux-system/         # Flux bootstrap manifests
│       ├── application/         # Flux Kustomization pointing to applications/overlays/<env>
│       └── infrastructure/      # Flux Kustomization pointing to infrastructure/overlays/<env>
├── bin/                         # Locally downloaded binaries (kustomize, kubeval)
├── Makefile                     # Automation for validation, linting, and build
└── README.md
```

---

### GitOps Flow Overview

1. **Flux bootstraps** via `clusters/<env>/flux-system/gotk-sync.yaml`
2. Flux applies root-level `Kustomization` resources:

    * `clusters/<env>/infrastructure/infra-kustomization.yaml`
    * `clusters/<env>/application/app-kustomization.yaml`
3. These point to overlays:

    * `infrastructure/overlays/<env>`
    * `applications/overlays/<env>`
4. Each overlay applies `base/` resources with environment-specific namespace and prefixes

---

### Makefile Commands

```bash
make install-tools         # Downloads kustomize + kubeval to ./bin
make build                 # Build manifests for both infra & apps
make validate              # Validate structure via Kustomize
make lint                  # Lint manifests with kubeval
make diff                  # Diff vs live cluster (requires kubectl)

make build ENV=production  # Run for a specific environment
```

---

### Infrastructure Components Managed

| Component                | Path                                 | Managed via |
| ------------------------ | ------------------------------------ | ----------- |
| CNPG (Postgres Operator) | `infrastructure/base/cnpg/`          | Helm        |
| Velero                   | `infrastructure/base/velero/`        | Helm        |
| CertManager              | `infrastructure/base/cert-manager/`  | Helm        |
| Ingress-NGINX            | `infrastructure/base/ingress-nginx/` | Helm        |

Environment-specific `PostgreSQL` clusters are stored in:

```bash
applications/base/postgres/
```

---

### Linting & Validation

* Kustomize is used to validate environment overlays
* `kubeval` is used with `--ignore-missing-schemas` to skip CRDs like HelmRelease
* For full validation, use:

```bash
make lint ENV=staging
```

### ✅ TODO

* [ ] Add HelmRelease for backend/frontend
* [ ] Enable CI with GitHub Actions for dry-run validation
* [ ] Integrate SOPS for secrets management
* [ ] Automate namespace creation per environment

---

Happy GitOps 🚀
