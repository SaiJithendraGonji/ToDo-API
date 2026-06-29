# ✅ ToDo REST API

> A lightweight REST API demonstrating CI/CD pipeline integration, container security scanning, Kubernetes deployment, and production-grade observability — using a simple task management application as the vehicle.

The application itself is intentionally simple. The point is everything around it: how it's built, secured, deployed, and monitored.

---

## 🏗️ What This Demonstrates

```
Code Push
    │
    ▼
GitHub Actions CI
    ├── Unit tests
    ├── SonarQube SAST scan
    ├── Trivy container scan (blocks on CRITICAL CVEs)
    └── Build + push to ECR/ACR
          │
          ▼
    Kubernetes Deploy (EKS / AKS)
          ├── Helm chart deployment
          ├── Vault secret injection (DB credentials)
          ├── HPA configured (scale on request rate)
          └── Prometheus metrics exposed (/metrics endpoint)
                │
                ▼
          Grafana Dashboard
                └── Request rate, error rate, latency (RED metrics)
```

---

## 📖 API Endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `GET` | `/health` | Liveness probe | None |
| `GET` | `/ready` | Readiness probe | None |
| `GET` | `/metrics` | Prometheus metrics | None |
| `GET` | `/tasks` | List all tasks | Bearer token |
| `GET` | `/tasks/:id` | Get task by ID | Bearer token |
| `POST` | `/tasks` | Create a task | Bearer token |
| `PUT` | `/tasks/:id` | Update a task | Bearer token |
| `DELETE` | `/tasks/:id` | Delete a task | Bearer token |

---

## 🔒 Security Implementation

- **Container hardening** — non-root user (UID 1000), read-only root filesystem, dropped all Linux capabilities
- **No secrets in images** — database credentials injected at runtime via Vault agent sidecar
- **Trivy scanning** — CI pipeline blocks deployment if CRITICAL vulnerabilities found in image
- **RBAC** — Kubernetes ServiceAccount with minimum required permissions only
- **Network Policy** — pod only accepts traffic from ALB ingress and Prometheus scraper
- **Distroless base image** — minimal attack surface, no shell in production image

---

## 📁 Repository Structure

```
├── src/                          # Application source code
│   ├── handlers/                 # Route handlers
│   ├── middleware/               # Auth, logging, metrics middleware
│   └── models/                   # Data models
├── docker/
│   └── Dockerfile                # Multi-stage build → distroless runtime
├── kubernetes/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── hpa.yaml
│   │   ├── networkpolicy.yaml
│   │   └── serviceaccount.yaml
│   └── overlays/
│       ├── dev/
│       └── production/
├── helm/
│   └── todo-api/                 # Helm chart (uses microservice-base pattern)
├── monitoring/
│   └── grafana-dashboard.json    # RED metrics dashboard
└── .github/
    └── workflows/
        ├── ci.yml                # Test + scan + build
        └── deploy.yml            # Helm deploy to EKS/AKS
```

---

## 🚀 Local Development

```bash
# Run locally
docker build -t todo-api:local .
docker run -p 8080:8080 \
  -e DB_HOST=localhost \
  -e DB_PORT=5432 \
  todo-api:local

# Run tests
npm test

# Scan image locally
trivy image todo-api:local --severity HIGH,CRITICAL

# Deploy to Kubernetes
helm install todo-api ./helm/todo-api/ \
  -f helm/todo-api/values-dev.yaml \
  --namespace apps
```

---

## 📊 Observability

The API exposes Prometheus metrics at `/metrics`:

```
# Request rate
http_requests_total{method, endpoint, status}

# Latency
http_request_duration_seconds{method, endpoint, quantile}

# Error rate
http_errors_total{method, endpoint, error_type}
```

Grafana dashboard included in `/monitoring/grafana-dashboard.json` — import directly into any Grafana instance connected to Prometheus.

---

## 🛠️ Tech Stack

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat&logo=helm&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
![HashiCorp Vault](https://img.shields.io/badge/Vault-FFEC6E?style=flat&logo=vault&logoColor=black)

---

## 📖 Related

- [HELM-Charts](https://github.com/SaiJithendraGonji/HELM-Charts) — The `microservice-base` chart this app uses
- [ENDTOEND_DEVSECOPS](https://github.com/SaiJithendraGonji/ENDTOEND_DEVSECOPS) — Full security pipeline this app runs through
- [CKA-Preparation](https://github.com/SaiJithendraGonji/CKA-Preparation) — Kubernetes concepts behind the deployment manifests
