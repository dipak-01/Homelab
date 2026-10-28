# 🏡 Dk's Homelab

[![Website](https://img.shields.io/badge/Dashboard-selfhost.dipakkurkute.dev-blue?style=flat-square)](https://selfhost.dipakkurkute.dev)
[![Uptime Status](https://img.shields.io/badge/Status%20Page-uptime.dipakkurkute.dev-brightgreen?style=flat-square)](https://uptime.dipakkurkute.dev/status/all)

> 🔗 **Quick Links:**
> - 🌐 **Website:** [https://selfhost.dipakkurkute.dev](https://selfhost.dipakkurkute.dev)
> - 📊 **Status Page:** [https://uptime.dipakkurkute.dev/status/all](https://uptime.dipakkurkute.dev/status/all)

A curated showcase of my personal homelab infrastructure, self-hosted services, container architecture, and fully declarative **GitOps** pipeline powered by **K3s (Kubernetes)**, **Helm**, **ArgoCD**, and **Bitnami Sealed Secrets**.

---

## 📌 Overview

This repository documents the architecture and configurations powering my personal homelab. Designed for privacy, reliability, and self-reliance, the environment hosts daily-driver applications ranging from AI-powered media backups to document processing, digital knowledge bases, and continuous health monitoring.

All core Kubernetes workloads are deployed declaratively and reconciled continuously via **GitOps** using **ArgoCD**.

### 🔄 Project Evolution & Phases
- **Phase 1 (Complete):** Docker Compose baseline stacks with isolated bridge networks and bind mounts.
- **Phase 2 (Complete):** Modular **Helm charts**, K3s native **Traefik Ingress**, and persistent storage with `Retain` policies.
- **Phase 3 (Active Phase):** **GitOps Continuous Delivery with ArgoCD** and zero-trust secret encryption using **Bitnami Sealed Secrets**.
- **Phase 4 (Upcoming):** Bare-metal and VM host bootstrapping & OS hardening with **Ansible**.

---

## 🏗️ Architecture & GitOps Flow

```
                               ┌───────────────────────────┐
                               │     *.dipakkurkute.dev    │
                               │   Tailscale Mesh Network  │
                               └─────────────┬─────────────┘
                                             │
                                             ▼
                               ┌───────────────────────────┐
                               │   Traefik Ingress / K3s   │
                               │   Subdomain HTTP Routing  │
                               └─────────────┬─────────────┘
                                             │
      ┌───────────────────┬──────────────────┼───────────────────┬───────────────────┐
      ▼                   ▼                  ▼                   ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   Homepage   │   │Paperless-ngx │   │    Memos     │   │    Immich    │   │    Faved     │
│  Dashboard   │   │ Document OCR │   │ Markdown DB  │   │ Media Cloud  │   │  Bookmarks   │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
                                             ▲
                                             │  (Reconciles & Deploys)
                               ┌─────────────┴─────────────┐
                               │          ArgoCD           │
                               │   Automated GitOps Sync   │
                               └─────────────▲─────────────┘
                                             │
                                             │  (Tracks main branch)
                               ┌─────────────┴─────────────┐
                               │    GitHub Repository      │
                               │  (Helm + SealedSecrets)   │
                               └───────────────────────────┘
```

---

## 🚀 Services & Deployment Status

| Service | Category | Endpoint | Deployment Mode | GitOps (ArgoCD) | Description |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **[Homepage](k3s/helm/homepage)** | Unified Dashboard | `homepage.dipakkurkute.dev` | 🟢 **K3s / Helm** | `Healthy` / `Synced` | Single pane of glass for all homelab services, real-time service health, resource widgets, and quick bookmarks. |
| **[Immich](k3s/helm/immich)** | Media Cloud | `immich.dipakkurkute.dev` | 🟢 **K3s / Helm** | `Healthy` / `Synced` | Google Photos alternative with ML facial recognition, VectorChord vector search, Valkey cache, and dedicated Postgres storage. |
| **[Paperless-ngx](k3s/helm/paperless)** | Document Archive | `paperless.dipakkurkute.dev` | 🟢 **K3s / Helm** | `Healthy` / `Synced` | Automated OCR pipeline with companion services (Valkey 9, Gotenberg, Tika) and persistent storage (`Retain` policy). |
| **[Memos](k3s/helm/memos)** | Knowledge Base | `memos.dipakkurkute.dev` | 🟢 **K3s / Helm** | `Healthy` / `Synced` | Privacy-first, lightweight micro-blogging and ephemeral note-taking service formatted in Markdown. |
| **[Faved](k3s/helm/faved)** | Bookmark Manager | `faved.dipakkurkute.dev` | 🟡 **K3s / Compose** | *In Progress* | Self-hosted web bookmark manager for organizing links and categorized resources. |
| **[Uptime Kuma](compose/uptime-kuma)** | Observability | `uptime.dipakkurkute.dev` | 🟢 **Docker Compose** | *Standalone* | Real-time monitoring suite tracking HTTP/HTTPS responsiveness, container status, and ping latency. |

*Legend: 🟢 Migrated & Running &nbsp;|&nbsp; 🟡 Staged / In Migration*

---

## 🐙 GitOps with ArgoCD

Cluster state is driven entirely by Git. When changes are pushed to `main`, ArgoCD automatically synchronizes the cluster to match the repository.

Application manifests are located under [`k3s/argocd/`](k3s/argocd/):

```text
k3s/argocd/
├── homepage-application.yaml     # ArgoCD Application: Homepage dashboard
├── immich-application.yaml       # ArgoCD Application: Immich photo suite
├── memos-application.yaml        # ArgoCD Application: Memos knowledge base
└── paperless-application.yaml    # ArgoCD Application: Paperless-ngx OCR stack
```

### GitOps Sync Policy
Each `Application` resource is configured with automated reconciliation and drift correction:
- **`automated.selfHeal: true`**: Automatically reverts any manual or out-of-band cluster modifications back to the Git source of truth.
- **`syncOptions: CreateNamespace=true`**: Automatically creates workload namespaces (`homepage`, `immich`, `paperless`, `memos`) on first deployment.
- **`prune: false`**: Safeguards stateful workloads and persistent claims against accidental deletion during initial transitions.

To apply all ArgoCD applications to the cluster:
```bash
kubectl apply -f k3s/argocd/
```

---

## 🔐 Zero-Trust Secrets (Bitnami Sealed Secrets)

To maintain a true GitOps workflow where all Kubernetes manifests can safely live in a public or shared Git repository, credentials are encrypted using **Bitnami Sealed Secrets**.

- Sensitive values (`API keys`, `DB passwords`, `Secret Keys`) are encrypted client-side using `kubeseal` and the cluster's asymmetric public key.
- The resulting `SealedSecret` custom resources can be safely committed to Git.
- The in-cluster `sealed-secrets-controller` decrypts them into native Kubernetes `Secret` objects at runtime.

Encrypted manifests live directly alongside their respective Helm charts:
```text
k3s/helm/
├── homepage/templates/secrets/homepage-sealedsecret.yaml
├── immich/templates/secrets/immich-sealedsecret.yaml
├── memos/templates/secrets/memos-sealedsecret.yaml
└── paperless/templates/secrets/paperless-sealedsecret.yaml
```

Unencrypted secrets (`k3s/secrets/*.yaml`) are explicitly excluded via `.gitignore`.

---

## ☸️ K3s Cluster & Storage Architecture

- **Ingress Controller:** K3s built-in **Traefik** (`ingressClassName: traefik`) handling subdomain routing, SSL termination, and ClusterIP routing.
- **Storage Strategy (`local-path-retain`):**
  - Host-local dynamic storage managed by Rancher Local Path Provisioner.
  - Critical stateful volumes (Immich photos & database, Paperless document library) use `reclaimPolicy: Retain` to ensure persistent storage is preserved across Helm releases, pod lifecycles, and cluster maintenance.

---

## 🌐 Networking & Access Model

- **Subdomain Routing:** Clean routing under `*.dipakkurkute.dev`.
- **Zero Public Exposure:** Secured entirely over a private **Tailscale** overlay network. Services are accessible exclusively to authenticated Tailnet devices.
- **Service Discovery:** Core intra-cluster services communicate over Kubernetes internal DNS (`<service>.<namespace>.svc.cluster.local`).

---

## 🤖 Ansible Automation Framework

> **Status:** 🚧 *Host Provisioning Baseline*

The [`ansible/`](ansible/) directory houses the infrastructure-as-code automation used for host bootstrapping, runtime setup, and node configuration prior to K3s cluster join.

```text
ansible/
├── inventory.yml          # Host definitions and node groups
├── playbook.yml           # Master host orchestration playbook
└── tasks/
    └── docker.yml         # Container runtime & Docker setup
```

---

## 📂 Repository Layout

```text
Homelab/
├── README.md                      # Homelab architecture & GitOps documentation
├── k3s/                           # Kubernetes (K3s) manifests & Helm charts
│   ├── argocd/                    # ArgoCD Application CRDs (GitOps declarations)
│   │   ├── homepage-application.yaml
│   │   ├── immich-application.yaml
│   │   ├── memos-application.yaml
│   │   └── paperless-application.yaml
│   ├── helm/                      # Service Helm charts
│   │   ├── faved/                 # Faved bookmark manager chart
│   │   ├── homepage/              # Homepage dashboard chart + sealed secrets
│   │   ├── immich/                # Immich media platform chart + sealed secrets
│   │   ├── memos/                 # Memos note-taking chart + sealed secrets
│   │   └── paperless/             # Paperless-ngx stack chart + sealed secrets
│   ├── storage/                   # StorageClass definitions (local-path-retain)
│   └── secrets/                   # Local secrets directory (git-ignored)
├── compose/                       # Docker Compose stacks (legacy / transition)
│   ├── faved/                     # Faved bookmark service
│   ├── homepage/                  # Homepage dashboard config
│   ├── immich-app/                # Immich media storage stack
│   ├── memos/                     # Note-taking service
│   ├── paperless-ngx/             # Document OCR stack
│   └── uptime-kuma/               # Uptime monitoring suite
└── ansible/                       # Host provisioning & automation playbooks
    ├── inventory.yml              # Target hosts and server groups
    ├── playbook.yml               # Main execution playbook
    └── tasks/
        └── docker.yml             # Docker installation & daemon setup
```
