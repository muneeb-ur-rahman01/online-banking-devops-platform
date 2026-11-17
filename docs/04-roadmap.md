# 04 · Phase-by-Phase Roadmap

Guiding principle: **local first.** Compose → Jenkins → security gates, and only
then AWS. Each phase ends with a **validation checklist** that must pass before
the next phase starts.

| # | Phase | Main outputs | Exit criteria (validated, not assumed) |
|---|---|---|---|
| 1 | Architecture + GitHub | Folder structure, architecture docs, diagrams, repo hygiene files, first commit on GitHub | Repo pushed; `.gitignore` verified to ignore `.env`/state |
| 2 | Microservices | 5 TypeScript Express services, health endpoints, unit + API tests, Postgres schemas and migrations needed by each service | `npm test` passes per service; services run locally against a Postgres container |
| 3 | React frontend | Dashboard, auth pages, accounts, transfer, history, beneficiaries, profile, notifications, admin | Full flows work in the browser against the local backend |
| 4 | Database hardening | Final migrations, seed data, constraints/indexes review, transfer consistency + idempotency tests | Concurrency tests prove no double debit and no negative balances |
| 5 | Docker | Multi-stage non-root Dockerfiles, `.dockerignore`, healthchecks, Compose for the whole stack | `docker compose up` gives a fully working app; images run as non-root |
| 6 | Jenkins CI/CD | Jenkins in Docker, Jenkinsfile + shared library, change detection, test reports, webhook | Real pipeline run with screenshots / console logs captured |
| 7 | DevSecOps | SonarQube + quality gate, Trivy (image/fs/config), npm audit, Gitleaks, Checkov | Pipeline **fails** on a deliberately introduced critical issue, then passes after the fix |
| 8 | Terraform + AWS | Bootstrap (S3 state), VPC, EKS, RDS, ECR, IAM, Secrets Manager, CloudWatch | `terraform plan` reviewed; apply **only with your approval**; resources verified |
| 9 | Kubernetes / EKS | Kustomize base + overlays, probes, HPA, PDB, NetworkPolicies, Ingress/ALB, External Secrets | `kubectl rollout status` OK; app reachable over HTTPS; rollback demonstrated |
| 10 | Monitoring + logging | kube-prometheus-stack, ServiceMonitors, 4 Grafana dashboards, alerts, Fluent Bit → CloudWatch | Dashboards show live data; a test alert fires |
| 11 | Final validation | E2E (Playwright), k6 load test, security review, final README with evidence | All 11 test scenarios pass; evidence captured from real runs |

## Adjustments to the original plan (and why)

1. **Schemas move partly into Phase 2.** Transfers cannot be implemented safely
   without real transactions and row locks. Each service gets its migrations
   while it is being built. Phase 4 then hardens them, adds seed data and adds
   concurrency tests.
2. **Phase 2 is split into sub-steps**, one service per step, each validated:
   2a shared conventions + auth-service → 2b account-service → 2c transaction-service →
   2d notification-service + RabbitMQ → 2e api-gateway.
3. **Kubernetes manifests use Kustomize** (`k8s/base` + `k8s/overlays/{dev,staging,prod}`)
   to satisfy rule 18 (separate environment configuration).
4. **Terraform uses per-environment root modules** (`terraform/environments/<env>`)
   that call shared modules, plus a one-time `terraform/bootstrap` for the state bucket.
5. **No shared runtime library between services.** Small amounts of duplicated
   boilerplate (logger, error handler) are an accepted cost of true independent
   deployability.
