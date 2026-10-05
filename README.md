# 🏡 Dk's Homelab

A curated showcase of my personal homelab infrastructure, self-hosted services, container architecture, and upcoming automation tooling.

---

## 📌 Overview

This repository documents the architecture and configurations powering my personal homelab. Designed for privacy, reliability, and self-reliance, the environment hosts daily-driver applications ranging from AI-powered media backups to document processing and continuous health monitoring.

All services are containerized using **Docker Compose** for isolation and reproducibility, while host-level provisioning and orchestration are architected to be driven by **Ansible**.

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
                          │     Homepage Dashboard    │
                          │        (Port 3000)        │
                          └─────────────┬─────────────┘
                                        │
        ┌───────────────────┬───────────┴───────────┬───────────────────┐
        ▼                   ▼                       ▼                   ▼
┌──────────────┐    ┌──────────────┐        ┌──────────────┐    ┌──────────────┐
│    Immich    │    │Paperless-ngx │        │    Memos     │    │ Uptime Kuma  │
│ Photo/Video  │    │Document Mgmt │        │ Markdown DB  │    │  Monitoring  │
│ (Port 2283)  │    │ (Port 8010)  │        │ (Port 5230)  │    │ (Port 3001)  │
└──────────────┘    └──────────────┘        └──────────────┘    └──────────────┘
```

---

## 🚀 Featured Services

| Service | Category | Endpoint / Port | Key Components | Description |
| :--- | :--- | :--- | :--- | :--- |
| **[Homepage](compose/homepage)** | Unified Dashboard | `homepage.dipakkurkute.dev` <br>`:3000` | Node.js, Custom CSS, Docker Socket | Single pane of glass for all homelab services, real-time container states, resource widgets, and quick-access bookmarks. |
| **[Immich](compose/immich-app)** | Media Cloud | `immich.dipakkurkute.dev` <br>`:2283` | PostgreSQL (VectorChord), Valkey, ML container | Self-hosted Google Photos alternative offering high-performance photo/video backup, vector-based semantic search, and on-device facial recognition. |
| **[Paperless-ngx](compose/paperless-ngx)** | Document Archive | `paperless.dipakkurkute.dev` <br>`:8010` | Valkey, Gotenberg, Apache Tika | Automated OCR pipeline and digital file cabinet. Automatically consumes, indexes, and categorizes scanned bills, statements, and office files. |
| **[Memos](compose/memos)** | Knowledge Base | `memos.dipakkurkute.dev` <br>`:5230` | SQLite backend | Privacy-first, lightweight micro-blogging and ephemeral note-taking service formatted in Markdown. |
| **[Uptime Kuma](compose/uptime-kuma)** | Observability | `uptime.dipakkurkute.dev` <br>`:3001` | SQLite, Multi-network Bridge | Real-time monitoring suite tracking HTTP/HTTPS responsiveness, container status, and ping latency with cross-network bridges into other stacks. |

---

## 🌐 Networking & Access Model

- **Domain Routing:** Services are mapped to custom subdomain routing (`*.dipakkurkute.dev`).
- **Secure Mesh Access:** Remote and node-to-node management is secured over a **Tailscale** overlay network, minimizing public port exposure.
- **Docker Network Isolation:** Stacks reside on isolated bridge networks (`homepage_default`, `immich_default`, `paperless_default`). **Uptime Kuma** bridges into these networks to monitor internal container health directly by container hostname.

---

## 🤖 Ansible Automation Framework

> **Status:** 🚧 *Architected & Under Active Implementation*

To transition from manual server setup to a fully reproducible **Infrastructure as Code (IaC)** model, the [`ansible/`](ansible/) directory establishes the automation blueprint for the homelab nodes.

```text
ansible/
├── inventory.yml          # Host definitions, node groups (VMs, bare-metal nodes)
├── playbook.yml           # Master orchestration playbook
└── tasks/
    └── docker.yml         # Automated Docker Engine & Compose installation
```

### Planned Automation Scope:

1. **Host Bootstrapping & Hardening:**
   - Automate bare-metal and VM baselining: base packages, non-root administrative users, SSH key injection, timezone, and locale.
   - Security hardening: automated security updates, SSH policy enforcement, and UFW firewall configuration.

2. **Docker Environment Provisioning ([`ansible/tasks/docker.yml`](ansible/tasks/docker.yml)):**
   - Automatically configure official upstream Docker repositories.
   - Install Docker Engine, containerd, and Docker Compose plugin.
   - Setup Docker user permissions, enable systemd daemons, and tune container log-rotation parameters.

3. **Declarative Stack Management:**
   - Orchestrate deployments and rolling updates of the compose services directly through Ansible plays.
   - Centralize secrets and sensitive tokens using **Ansible Vault** for encrypted, version-controlled credential management.

---

## 📂 Repository Layout

```text
Homelab/
├── README.md                      # Homelab showcase & architectural documentation
├── compose/                       # Service definitions and persistent configs
│   ├── homepage/                  # Dashboard layout, widgets, and service cards
│   ├── immich-app/                # Immich media storage and ML stack
│   ├── memos/                     # Note-taking service and SQLite volume
│   ├── paperless-ngx/             # Document OCR, Tika, and Gotenberg services
│   └── uptime-kuma/               # Uptime monitoring configuration
└── ansible/                       # Planned Infrastructure as Code automation
    ├── inventory.yml              # Target hosts and server groups
    ├── playbook.yml               # Main execution playbook
    └── tasks/
        └── docker.yml             # Docker installation & daemon setup
```
