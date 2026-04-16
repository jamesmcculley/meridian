# Meridian Software — SRE Platform

A portfolio-grade observability and security platform built to production standards. Meridian Software's primary product is **StageGrid** — a live events ticketing platform. The Meridian Platform is the SRE layer that runs it: a multi-cloud architecture with each cloud assigned a specific job, built incrementally from a working on-premises foundation.

> Built by [James McCulley](https://jamesmcculley.dev) — Senior SRE  
> GitHub: [github.com/jamesmcculley](https://github.com/jamesmcculley)

---

## Why This Project Exists

Most portfolio infrastructure projects deploy a generic app and call it done. This one is different. Meridian is built to answer a specific question: *what does a senior SRE's platform actually look like when it's designed for real operational complexity?*

StageGrid's traffic profile — sharp on-sale spikes, high checkout criticality, async notification fanout — forces real decisions about autoscaling, SLO design, failure domain isolation, and chaos engineering. Every tool in this stack earned its place.

---

## Architecture Overview

**Meridian Software** is the fictional company. **StageGrid** is the product — a live events ticketing and infrastructure platform. The Meridian Platform is the SRE layer that runs it.

The architecture assigns each cloud a specific job rather than duplicating infrastructure across providers. The on-premises stack is the working foundation today; cloud planes are being added in phases.

### Infrastructure

| Environment | Provider | Runtime | Status |
|---|---|---|---|
| meridian-onprem | On-premises | docker-compose | Running — current foundation |
| meridian-aws | AWS | k3s on EC2 t3.micro | Planned — security plane |
| meridian-gcp | GCP | k3s on e2-micro | Planned — observability plane |
| meridian-azure | Azure | k3s on B1s | Planned — identity / GitOps plane |

### StageGrid Services

StageGrid's service architecture covers five bounded domains. The first service to be implemented is `stagegrid-tickets` — a Python/FastAPI application handling seat reservation and checkout, instrumented with OpenTelemetry for distributed tracing. The remaining four services are planned and will follow.

| Service | Responsibility | SLO target | Status |
|---|---|---|---|
| stagegrid-tickets | Seat reservation and checkout | 99.9% | Building next |
| stagegrid-catalog | Event and venue browse | 99.5% | Planned |
| stagegrid-notify | Fan notification fanout | 99.0% | Planned |
| stagegrid-identity | Auth and access control | 99.9% | Planned |
| stagegrid-loadgen | Synthetic traffic generation | — | Planned |

### Observability Stack

| Pillar | Tool | Status |
|---|---|---|
| Metrics | VictoriaMetrics | Running |
| Logs | Quickwit | Running |
| Traces | Jaeger | Planned |
| Collection | Fluent Bit + Vector | Running |
| Dashboards | Grafana | Planned (GCP plane) |
| Instrumentation | OpenTelemetry | Planned |

### Security Stack

| Component | Purpose | Status |
|---|---|---|
| HashiCorp Vault | Secrets management, PKI, short-lived credentials | Running |
| Falco (eBPF) | Runtime syscall monitoring and anomaly detection | Planned |
| OPA / Gatekeeper | Kubernetes admission control policies | Planned |
| Trivy | Container image vulnerability scanning in CI | Planned |
| Linkerd mTLS | Encrypted, authenticated service-to-service communication | Planned |

### GitOps & Deployment

All manifests and Helm values are version-controlled in this repository. ArgoCD is the planned GitOps control plane (App of Apps pattern); it is not yet deployed.

---

## Python Tooling

Platform tooling is written in Python and lives in `tools/`. Only `meridian-core` is implemented; the remaining tools are planned.

| Tool | Description | Status |
|---|---|---|
| `meridian-core` | Shared library: config, Vault client, service discovery | Implemented |
| `logparse` | Log parsing and structured event extraction from Quickwit | Planned |
| `py-exporter` | Custom VictoriaMetrics exporter for StageGrid business metrics | Planned |
| `alert-router` | Alert routing and enrichment | Planned |
| `atlas-ops` | Operational automation: node drain, certificate rotation | Planned |
| `canary-analyzer` | Automated canary analysis using VictoriaMetrics metrics | Planned |

---

## Repository Structure

```
/
├── README.md
├── STRUCTURE.md           # Detailed directory and file reference
├── CHANGELOG.md
├── aws/
│   └── vault/config/      # Vault server config for AWS plane
├── onprem/
│   ├── docker-compose.yml # On-prem stack: Vault, VictoriaMetrics, Quickwit, Nginx, MongoDB
│   ├── nginx/             # TLS reverse proxy config
│   ├── node-exporter/     # Node Exporter TLS config
│   └── vault/config/      # Vault server config for on-prem
├── observability/
│   ├── victoriametrics/   # Scrape config
│   ├── quickwit/          # Index and ingest config
│   └── otel/              # Fluent Bit and Vector pipeline configs
├── security/
│   └── README.md          # Security tooling overview
├── gitops/
│   └── helm/meridian-chart/ # Helm chart scaffold
└── tools/
    └── meridian-core/     # Core Python library — implemented
```

---

## SLO Design

SLO targets are defined for each StageGrid service. Once services are deployed, burn rate alerting will fire at 1-hour and 6-hour windows tracked in VictoriaMetrics.

| Service | Target | 30-Day Error Budget |
|---|---|---|
| stagegrid-tickets | 99.9% availability | 43.2 minutes |
| stagegrid-catalog | 99.5% availability | 3.6 hours |
| stagegrid-notify | 99.0% availability | 7.2 hours |
| stagegrid-identity | 99.9% availability | 43.2 minutes |

---

## Compliance Design Targets

- **PCI-DSS** — stagegrid-tickets will be PCI-scoped. Vault is designed to manage credentials with short-lived leases and full audit logging. Network policies will isolate PCI workloads to dedicated namespaces.
- **SOC2 Type II** — Access control, audit logging, and change management controls are mapped to ArgoCD (change management) and Vault (access control); implementation follows service deployment.

---

## Contact

**James McCulley** — Senior Site Reliability Engineer  
[jamesmcculley.dev](https://jamesmcculley.dev) · [github.com/jamesmcculley](https://github.com/jamesmcculley)



## Current Status

> Last updated: April 2026

### Running

All services below run via docker-compose in the `onprem/` configuration with TLS throughout.

- **Vault** — initialized and unsealed (Shamir unseal, file storage backend); KMS auto-unseal planned
- **VictoriaMetrics** — deployed, ingesting metrics via Node Exporter
- **Quickwit** — deployed for log indexing
- **Vector / Fluent Bit** — log collection and shipping pipeline configured end-to-end
- **Nginx** — TLS reverse proxy
- **MongoDB** — running via docker-compose

### Roadmap

The following are planned but not yet deployed:

- **GCP observability plane** — migrate VictoriaMetrics, Quickwit, and Grafana to a GCP e2-micro (permanently free tier); next active work item
- **Falco** — eBPF runtime security monitoring
- **OPA/Gatekeeper** — Kubernetes admission control policies
- **ArgoCD** — GitOps control plane, App of Apps pattern
- **Linkerd** — service mesh with mTLS across all services
- **KMS auto-unseal** — Vault auto-unseal via AWS KMS, eliminating manual operator unseal

### Intended Architecture

The table below describes the target multi-cloud placement. It is an architecture target, not a description of current state.

| Plane | Cloud | Intended components |
|---|---|---|
| Security | AWS | Vault (KMS auto-unseal), Falco, GuardDuty, OPA |
| Observability | GCP | VictoriaMetrics, Quickwit, Grafana |
| Identity / GitOps | Azure | Entra ID, ArgoCD, Linkerd |
| Edge / dev | On-prem | docker-compose stack (current), k3s (planned) |
