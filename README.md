# 🏡 Dk's Homelab

A curated showcase of my personal homelab infrastructure, self-hosted services, container architecture, and ongoing migration to a lightweight **K3s (Kubernetes)** cluster orchestrated via **Helm**.

---

## 📌 Overview

This repository documents the architecture and configurations powering my personal homelab. Designed for privacy, reliability, and self-reliance, the environment hosts daily-driver applications ranging from AI-powered media backups to document processing, digital knowledge bases, and continuous health monitoring.

### 🔄 Project Phase: Migration to K3s & Helm
The homelab is actively transitioning from standalone **Docker Compose** stacks to a modular, declarative **K3s Kubernetes** cluster:
- **Phase 1 (Complete):** Docker Compose baseline stacks with isolated bridge networks and bind mounts.
- **Phase 2 (Active Phase):** Migration of core services to **K3s** managed via custom **Helm charts**, **Traefik Ingress**, **PersistentVolumeClaims (PVCs)**, and structured secret injection.
- **Phase 3 (Upcoming):** Host provisioning automation with **Ansible** and GitOps-driven cluster deployments.

---

## 🏗️ Architecture & Stack

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
        ┌───────────────────┬───────────┴───────────┬───────────────────┬───────────────────┐
        ▼                   ▼                       ▼                   ▼                   ▼
 ┌──────────────┐    ┌──────────────┐        ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
 │   Homepage   │    │Paperless-ngx │        │    Memos     │    │    Faved     │    │    Immich    │
 │  Dashboard   │    │ Document OCR │        │ Markdown DB  │    │  Bookmarks   │    │ Photo/Video  │
 │ (ClusterIP)  │    │ (PVCs + Val) │        │ (ClusterIP)  │    │ (ClusterIP)  │    │ (Compose/K8s)│
 └──────────────┘    └──────────────┘        └──────────────┘    └──────────────┘    └──────────────┘
                                        │
                                        ▼
                             ┌─────────────────────┐
                             │     Uptime Kuma     │
                             │  Health & Latency   │
                             │ (Cross-stack Probe) │
                             └─────────────────────┘
```

---

## 🚀 Services & Migration Status

| Service | Category | Endpoint | Stack / Mode | Key Components | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **[Homepage](k3s/helm/homepage)** | Unified Dashboard | `homepage.dipakkurkute.dev` | 🟢 **K3s / Helm** | Node.js, Custom CSS, Traefik Ingress | Single pane of glass for all homelab services, real-time service health, resource widgets, and quick bookmarks. |
| **[Paperless-ngx](k3s/helm/paperless)** | Document Archive | `paperless.dipakkurkute.dev` | 🟢 **K3s / Helm** | Valkey broker, Gotenberg, Apache Tika, 5x PVCs | Automated OCR pipeline and digital file cabinet with dedicated PVCs for data, media, export, consume, and broker storage. |
| **[Memos](k3s/helm/memos)** | Knowledge Base | `memos.dipakkurkute.dev` | 🟢 **K3s / Helm** | SQLite backend, hostPath volume, Traefik | Privacy-first, lightweight micro-blogging and ephemeral note-taking service formatted in Markdown. |
| **[Faved](k3s/helm/faved)** | Bookmark Manager | `faved.dipakkurkute.dev` | 🟡 **K3s / Compose** | Apache/PHP, SQLite/storage | Modern, self-hosted web bookmark manager for organizing links and resources. |
| **[Immich](compose/immich-app)** | Media Cloud | `immich.dipakkurkute.dev` | 🟡 **Compose / Helm** | PostgreSQL (VectorChord), Valkey, ML container | Self-hosted Google Photos alternative offering high-performance photo/video backup, vector-based semantic search, and facial recognition. |
| **[Uptime Kuma](compose/uptime-kuma)** | Observability | `uptime.dipakkurkute.dev` | 🟢 **Docker Compose** | SQLite, Multi-network Bridge | Real-time monitoring suite tracking HTTP/HTTPS responsiveness, container status, and ping latency across network bridges. |

*Legend: 🟢 Migrated / Running in Target State &nbsp;|&nbsp; 🟡 Helm Chart Scaffolded / Parallel Compose Operation*

---

## ☸️ K3s & Helm Architecture

Workloads migrated to Kubernetes are located under [`k3s/`](k3s/) and structured into modular Helm charts and secret templates:

```text
k3s/
├── helm/
│   ├── homepage/             # Helm chart: Homepage dashboard with hostPath mounts
│   ├── paperless/            # Helm chart: Multi-container pod (Web, Valkey, Gotenberg, Tika) + PVCs
│   ├── memos/                # Helm chart: Memos deployment with SecretKeyRef configuration
│   ├── faved/                # Helm chart: Faved bookmark manager deployment & ingress
│   └── immich/               # Helm chart: Immich deployment templates & configuration
└── secrets/                  # Workload secrets (git-tracked templates & local credentials)
    ├── faved.yaml
    ├── homepage.yaml
    ├── immich.yaml
    ├── memos.yaml
    └── paperless.yaml
```

### Key Highlights:
- **Paperless-ngx Stateful Architecture:** Configured with dedicated `PersistentVolumeClaim` definitions (`paperless-data`, `paperless-media`, `paperless-export`, `paperless-consume`, `paperless-redis`) and companion services (Valkey 9, Gotenberg 8, Apache Tika 3).
- **Ingress Layer:** Managed using K3s native **Traefik** ingress controller (`ingressClassName: traefik`) mapping subdomain routes cleanly to Kubernetes `ClusterIP` services.
- **Persistent Storage:** Combination of `hostPath` mappings for shared host directories (`/home/dk/Homelab_Data/`) and Kubernetes PVCs.

---

## 🌐 Networking & Access Model

- **Domain Routing:** Services are exposed via clean subdomain routing under `*.dipakkurkute.dev`.
- **Mesh Security:** Access is secured over a private **Tailscale** overlay network, keeping services inaccessible from the public internet without explicit exposure.
- **Internal Service Discovery:** Kubernetes internal DNS (`<service>.<namespace>.svc.cluster.local`) handles pod-to-pod communication, while legacy Compose networks retain bridge-level isolation.

---

## 🤖 Ansible Automation Framework

> **Status:** 🚧 *Scaffolded for Host Provisioning*

The [`ansible/`](ansible/) directory contains the infrastructure-as-code baseline to configure nodes prior to K3s cluster join and container runtime setup.

```text
ansible/
├── inventory.yml          # Host definitions and node groups (VMs, bare-metal nodes)
├── playbook.yml           # Master host orchestration playbook
└── tasks/
    └── docker.yml         # Automated Docker Engine & containerd runtime installation
```

### Planned Automation Scope:
1. **Host Bootstrapping & Hardening:** SSH policy enforcement, firewall rules (UFW), time synchronization, and baseline packages.
2. **Container & Node Runtimes:** Automated installation of Docker Engine, containerd, and K3s node prerequisites.
3. **Secrets Management:** Integration with Ansible Vault or sealed secrets for zero-exposure credential storage.

---

## 📂 Repository Layout

```text
Homelab/
├── README.md                      # Homelab architecture & status overview
├── k3s/                           # Kubernetes (K3s) manifests & Helm charts
│   ├── helm/                      # Service Helm charts
│   │   ├── faved/                 # Faved bookmark manager chart
│   │   ├── homepage/              # Homepage dashboard chart
│   │   ├── immich/                # Immich media platform chart
│   │   ├── memos/                 # Memos note-taking chart
│   │   └── paperless/             # Paperless-ngx stack chart & PVC manifests
│   └── secrets/                   # Workload secrets and env definitions
├── compose/                       # Docker Compose stacks (baseline / transition)
│   ├── faved/                     # Faved bookmark service
│   ├── homepage/                  # Homepage dashboard config and icons
│   ├── immich-app/                # Immich media storage and ML stack
│   ├── memos/                     # Note-taking service and SQLite volume
│   ├── paperless-ngx/             # Document OCR, Tika, and Gotenberg services
│   └── uptime-kuma/               # Uptime monitoring configuration
└── ansible/                       # Host provisioning & automation playbooks
    ├── inventory.yml              # Target hosts and server groups
    ├── playbook.yml               # Main execution playbook
    └── tasks/
        └── docker.yml             # Docker installation & daemon setup
```
