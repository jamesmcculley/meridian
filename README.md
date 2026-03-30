# Meridian

**Multi-cloud observability and security platform — actively under construction.**

Meridian is a portfolio project documenting the design and build of a production-grade SRE platform across AWS, GCP, Azure, and on-premises infrastructure. This README reflects the intended architecture. See the [Status](#status) section for what's actually running.

The goal is intentional cloud placement — each provider used for its genuine strengths rather than duplicating the same stack everywhere.

---

## Intended Architecture

### Design Philosophy

Every service is planned to live where it does for a specific reason. Multi-cloud is an architectural decision, not a checkbox.

| Plane | Provider | Rationale |
|---|---|---|
| Security & Secrets | AWS | KMS auto-unseal, IAM auth, GuardDuty |
| Observability | GCP | Permanently free compute, excellent data networking |
| Identity & GitOps | Azure | Entra ID Workload Identity, enterprise change management |
| Edge / Dev | On-prem (k3s) | Constrained environment simulation, zero cloud cost |

### Cluster Strategy

Managed Kubernetes (EKS/GKE/AKS) is intentionally avoided to keep lab costs under $10/month. Each cloud node will run k3s on a single VM — a deliberate cost optimization that mirrors what a cost-conscious team would do for non-production workloads. Production deployment notes are documented per-component in `docs/ARCHITECTURE.md`.

---

## Planned Stack

### AWS — Security Plane

- **Vault** — KMS auto-unseal, IAM auth backend (zero static credentials)
- **Falco** — runtime threat detection via eBPF
- **OPA/Gatekeeper** — policy enforcement across all clusters
- **Trivy** — container image scanning in CI
- **GuardDuty** — findings routed via EventBridge → `alert-router`
- **ECR** — container registry

Infrastructure target: k3s on EC2 t3.micro (~$8/mo, stopped when idle)

### GCP — Observability Plane

- **VictoriaMetrics** — long-term metrics storage, scraping all three clouds
- **Quickwit** — cost-optimized log indexing and search
- **Grafana** — single pane of glass, federating data from all providers
- **Fluent Bit / Vector** — log shipping from all clusters
- **OpenTelemetry Collector** — trace aggregation

Infrastructure target: e2-micro (permanently free in us-central1)

### Azure — Identity & GitOps Plane

- **Entra ID** — Workload Identity Federation, OIDC-based, no long-lived credentials
- **ArgoCD** — GitOps controller managing deployments across all three clusters
- **Linkerd** — service mesh with mTLS
- **Compliance dashboards** — SOC2/PCI-DSS control mapping

Infrastructure target: k3s on B1s VM (free for 12 months)

### On-Prem — Edge / Dev Plane

- **k3s** via OrbStack on MacBook Pro ← *starting here*
- **Jaeger** — distributed tracing UI
- **OpenTelemetry Collector** — local aggregation before shipping to GCP
- **canary-analyzer** — cross-cloud canary deployment and analysis

---

## Python Tooling

Planned tooling written in stdlib-preferred Python. Each tool will be written without AI assistance.

| Tool | Purpose |
|---|---|
| `meridian-core` | Shared config, auth helpers, cloud client abstractions |
| `logparse` | Log ingestion and normalization across providers |
| `py-exporter` | Custom Prometheus exporter for platform metrics |
| `alert-router` | Routes GuardDuty/Falco alerts to appropriate channels |
| `atlas-ops` | Operational runbook automation |
| `canary-analyzer` | Cross-cloud canary deployment and analysis |

---

## Security Goals

The target security posture: no long-lived credentials anywhere in the system.

- **AWS** — EC2 instance role only. Vault uses KMS auto-unseal + IAM auth.
- **GCP** — Workload Identity. No service account key files on disk.
- **Azure** — Workload Identity Federation via Entra ID OIDC.
- **Vault** — secrets authority. All dynamic credentials issued here with short TTLs.

---

## Target Cost

| Component | Monthly Cost |
|---|---|
| GCP e2-micro | Free (permanent) |
| Azure B1s | Free (12 months) |
| AWS t3.micro (on-demand, stopped when idle) | ~$0–8 |
| OrbStack | Free |
| **Total** | **< $10/mo** |

---

## Repository Structure

```
MERIDIAN/
├── aws/                  # Terraform, Vault config, Falco rules
├── gcp/                  # VictoriaMetrics, Quickwit, Grafana
├── azure/                # ArgoCD, Linkerd, Entra ID config
├── onprem/               # k3s manifests, Jaeger, OTel
├── python/
│   ├── meridian-core/
│   ├── logparse/
│   ├── py-exporter/
│   ├── alert-router/
│   ├── atlas-ops/
│   └── canary-analyzer/
├── dashboards/           # Grafana dashboard JSON
├── compliance/           # SOC2/PCI-DSS control mappings
├── docs/
│   ├── ARCHITECTURE.md
│   ├── OPERATIONS.md
│   └── TROUBLESHOOTING.md
└── README.md
```

---

## Status

Build is in progress. Starting with on-prem k3s and working outward.

| Component | Status |
|---|---|
| On-prem k3s (OrbStack) | 🟡 In progress |
| Python tooling (meridian-core, logparse) | 🟡 In progress |
| GCP observability plane | 🔲 Planned |
| AWS security plane | 🔲 Planned |
| Azure identity/GitOps plane | 🔲 Planned |
| Grafana dashboards | 🔲 Planned |
| Compliance mapping | 🔲 Planned |

Architecture decisions and rationale are documented ahead of implementation in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

---

## Author

James McCulley — [jamesmcculley.dev](https://jamesmcculley.dev) · [github.com/jamesmcculley](https://github.com/jamesmcculley)
