# 02 · Technology Stack and Workstation Setup

## 1. Technology stack and why each tool is used

| Layer | Tool | Why we use it |
|---|---|---|
| Language/runtime | **Node.js 24 LTS** + **TypeScript** | One language across frontend and backend. Type safety catches money/ID mistakes at compile time. LTS = long security support. |
| Backend framework | **Express 5** | Minimal, well-known, huge middleware ecosystem (helmet, rate-limit, proxy). |
| Frontend | **React + Vite** | Component UI, fast dev server, static build served by unprivileged nginx. |
| Validation | **zod** | One schema = runtime validation + TypeScript types. |
| Passwords | **argon2 (Argon2id)** | Current OWASP-recommended password hash. |
| Tokens | **jose** (RS256 JWT) | Standards-compliant JWT library; asymmetric keys mean only auth-service can sign. |
| Database | **PostgreSQL 17** | Real ACID transactions, row locks (`SELECT … FOR UPDATE`), CHECK constraints. Exactly what money needs. |
| Migrations | **node-pg-migrate** (SQL files) | Versioned, repeatable schema changes per service. |
| Messaging | **RabbitMQ 4** | Durable queues + dead-lettering for notification events. |
| Logging | **pino** | Fast structured JSON logs with built-in field redaction. CloudWatch-friendly. |
| Metrics | **prom-client** | Exposes `/metrics` for Prometheus. |
| Tests | **Vitest + Supertest + Testcontainers**, **Playwright** (E2E), **k6** (performance) | Unit/API tests, real-Postgres integration tests, browser E2E, load tests. |
| Version control | **Git + GitHub** | Source of truth, PRs, webhooks to Jenkins. |
| CI/CD | **Jenkins LTS** (declarative pipeline + shared library) | Required by the brief; the industry-standard self-hosted CI. |
| Code quality | **SonarQube Community Build** | Static analysis, coverage, quality gate that blocks the pipeline. |
| Security scanning | **Trivy**, **npm audit**, **Gitleaks**, **Checkov** | Image/filesystem/IaC/K8s scanning, dependency CVEs, secret detection, Terraform policy checks. |
| Containers | **Docker + Docker Compose** | Same artefact runs on a laptop and in EKS; Compose runs the whole stack locally. |
| Registry | **Amazon ECR** | Private registry next to EKS, IAM-controlled, scan-on-push, immutable tags. |
| Orchestration | **Kubernetes on Amazon EKS** + **Kustomize** | Managed control plane. Kustomize overlays give dev/staging/prod without templating. |
| IaC | **Terraform** (AWS provider) | Reproducible, reviewable infrastructure; S3 backend with native locking. |
| Ingress | **AWS Load Balancer Controller → ALB** | Kubernetes Ingress becomes a real ALB with ACM TLS. |
| Secrets | **AWS Secrets Manager + External Secrets Operator** (EKS Pod Identity) | Secrets never live in Git; pods get them as K8s Secrets synced from AWS. |
| Monitoring | **kube-prometheus-stack** (Prometheus, Alertmanager, Grafana) | Standard Kubernetes monitoring bundle. |
| Logging (cloud) | **Fluent Bit → CloudWatch Logs** | Centralised, searchable logs for every pod. |

> **Version policy:** every image tag, npm dependency, Helm chart and Terraform
> provider is pinned to an exact version **in the phase where it is introduced**,
> after checking the current stable release. `latest` is never used.

## 2. Workstation status (checked 2026-11-17)

| Tool | Found | Needed from | Action |
|---|---|---|---|
| Git | 2.49 ✅ | Phase 1 | — |
| Node.js | 22.15 ✅ | Phase 2 | Recommended: install **Node 24 LTS** (Node 22 leaves support in April 2027). |
| npm | 11.19 ✅ | Phase 2 | — |
| Docker Desktop + Compose | 29.4 / v5.1 ✅ | Phase 5 | — |
| kubectl | 1.34 ✅ | Phase 9 | — |
| WSL Ubuntu | installed ✅ | optional | Run `.sh` scripts from Ubuntu if preferred. |
| GitHub account | ? | Phase 1 | Create one if you do not have one. |
| GitHub CLI `gh` | ❌ | optional | `winget install --id GitHub.cli` |
| Terraform | ❌ | Phase 8 | `winget install --id Hashicorp.Terraform` |
| AWS CLI v2 | ❌ | Phase 8 | `winget install --id Amazon.AWSCLI` |
| Helm | ❌ | Phase 9 | `winget install --id Helm.Helm` |
| Trivy | ❌ | Phase 7 | Runs as a Docker container, so no install needed |
| Jenkins, SonarQube | ❌ | Phase 6–7 | Run as Docker containers locally, so no install needed |
| Java | ❌ | — | Not needed; Jenkins/SonarQube images ship their own JRE |
| psql | ❌ | optional | `docker exec` into the Postgres container instead |

Do **not** install the Phase 6–10 tools yet. We install each tool when we
reach its phase and verify it then.

## 3. Local-first principle

Everything up to and including Phase 7 runs **on your laptop at zero cloud
cost**:

| Cloud component | Free local equivalent |
|---|---|
| EKS | Docker Desktop Kubernetes or `kind` |
| RDS PostgreSQL | `postgres` container in Compose |
| ECR | Local Docker images (or a local `registry:2` container) |
| Secrets Manager | `.env` file (git-ignored) / plain K8s Secret on the local cluster |
| ALB | `ingress-nginx` on the local cluster |
| CloudWatch | `docker compose logs` / Grafana Loki (optional) |
| Jenkins/SonarQube servers | Docker containers on the laptop |
| GitHub webhook to a laptop | `smee.io` relay or `ngrok`, or SCM polling |
