# 03 · AWS Cost Considerations

> These prices are **approximate on-demand prices for us-east-1** and are meant
> for planning only. Prices change and differ by region. Confirm them with the
> [AWS Pricing Calculator](https://calculator.aws/) before Phase 8, and
> **set an AWS Budget alert before you create anything.**

## 1. What the dev environment costs

| Component | Dev configuration | ≈ per hour | ≈ per month (24×7) |
|---|---|---|---|
| EKS control plane | 1 cluster, standard support | $0.10 | ~$73 |
| EKS worker nodes | 2 × `t3.medium` on-demand (Spot is ~60–70% cheaper) | ~$0.083 | ~$61 |
| Node EBS volumes | 2 × 20 GB gp3 | — | ~$3 |
| NAT Gateway | **1** (not one per AZ) + data processing $0.045/GB | $0.045 | ~$33 + data |
| Application Load Balancer | 1 ALB shared by all services via Ingress | ~$0.0225 + LCU | ~$18–22 |
| RDS PostgreSQL | `db.t4g.micro`, single-AZ, 20 GB gp3, 7-day backups | ~$0.016 | ~$14 |
| Public IPv4 addresses | ALB + NAT (~3 IPs) at $0.005/h each | ~$0.015 | ~$11 |
| ECR | ~2 GB of images, with a lifecycle policy | — | ~$0.20 |
| Secrets Manager | ~6 secrets × $0.40 | — | ~$2.40 |
| CloudWatch Logs | ~2 GB ingested, 7-day retention | — | ~$1–2 |
| Prometheus storage | 10 GB gp3 PVC | — | ~$1 |
| S3 Terraform state | a few KB | — | ~$0 |
| **Total** | | **≈ $0.30/hour** | **≈ $220–250/month if left running** |

**The biggest costs are EKS ($73), nodes, NAT Gateway and ALB. They are charged
per hour whether or not anyone uses the app.**

## 2. Cost strategy for this project

1. **Build and test everything locally first** (Phases 2–7 cost $0).
2. In Phase 8+, work in **sessions**: `terraform apply` → deploy → demo / capture
   evidence → `terraform destroy`. A 4-hour session costs roughly **$1.20–$1.50**.
3. Run one NAT Gateway, not one per AZ (dev only, accepting less availability).
4. Use Spot instances for dev node groups.
5. Run RDS single-AZ with no Multi-AZ in dev. Production would use Multi-AZ.
6. Set CloudWatch log retention to 7 days and an ECR lifecycle policy that keeps the last 10 images.
7. Create an **AWS Budget** ($20–30) with email alerts before Phase 8.
8. New AWS accounts may receive Free Tier credits. Check your account's
   current Free Tier offer, because EKS itself is **not** free.

## 3. Production vs dev

| Setting | dev | staging | prod |
|---|---|---|---|
| NAT Gateways | 1 | 1 | 1 per AZ |
| RDS | t4g.micro single-AZ | t4g.small single-AZ | Multi-AZ, larger class, deletion protection |
| Nodes | 2 × t3.medium Spot | 2 × t3.medium | 3+ on-demand across AZs |
| Replicas per service | 1 | 2 | 2–3 + HPA |
| Log retention | 7 days | 14 days | 90+ days |

## 4. Cleanup

Cleanup instructions (`kubectl delete ingress` first so the ALB is removed,
then `terraform destroy`, then verify in the console that no ALB, NAT, EIP or
RDS snapshot is left) will be documented in Phase 8. **No destructive Terraform
command will be run without your explicit approval.**
