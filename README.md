# mdfahim.dev — DevSecOps Portfolio Pipeline

> Personal portfolio site built with a production-grade DevSecOps pipeline.
> **Live:** [https://ec2.mdfahim.dev](https://ec2.mdfahim.dev)

---

## 🚀 Pipeline Overview

```
┌─────────────────────────────────────────────────────┐
│                   LOCAL MACHINE (Mac)                │
│                                                      │
│  index.html → dockerfile → nginx.conf               │
│  docker-compose.yml → .github/workflows/            │
└──────────────────┬──────────────────────────────────┘
                   │ git push origin staging
                   ↓
┌─────────────────────────────────────────────────────┐
│                    GITHUB                            │
│                                                      │
│  Repository: Fahim017803/mdfahim.dev                │
│  Branches: main (production) │ staging (test)       │
│                                                      │
│  ┌─────────── GitHub Actions ──────────────┐        │
│  │                                          │        │
│  │  PUSH triggers (parallel):               │        │
│  │  ├── Secret Scan (Gitleaks)              │        │
│  │  ├── Code Quality (HTMLHint)             │        │
│  │  ├── Docker Lint (Hadolint)              │        │
│  │  └── Dependency Scan (Trivy)            │        │
│  │            ↓ all pass                   │        │
│  │  Docker Build & Push                    │        │
│  │  → fahim017803/mdfahim.dev:latest       │        │
│  │  → fahim017803/mdfahim.dev:$sha         │        │
│  │            ↓                            │        │
│  │  Deploy Staging (auto, staging only)    │        │
│  │            ↓                            │        │
│  │  Deploy Production (manual approve)     │        │
│  │            ↓                            │        │
│  │  OWASP ZAP Security Scan               │        │
│  │  Lighthouse CI (Performance/SEO)        │        │
│  │                                          │        │
│  │  Every Monday 9AM UTC:                  │        │
│  │  ├── SSL Certificate Check             │        │
│  │  └── Broken Link Check                 │        │
│  └──────────────────────────────────────────┘        │
└──────────────────┬──────────────────────────────────┘
                   │
         ┌─────────┴─────────┐
         ↓                   ↓
┌────────────────┐   ┌───────────────────────────────┐
│   DOCKER HUB   │   │         AWS EC2 (Tokyo)        │
│                │   │                                │
│ fahim017803/   │   │  ┌─────────────────────────┐  │
│ mdfahim.dev:   │   │  │   Staging EC2            │  │
│  :latest  ←───┼───┼→ │   ip: 52.196.204.12      │  │
│  :$sha         │   │  │   Runner: My-porfolio-   │  │
│                │   │  │           staging        │  │
└────────────────┘   │  │   Docker: nginx:alpine   │  │
                     │  │   Port: 80 only          │  │
                     │  │   http://52.196.204.12   │  │
                     │  └─────────────────────────┘  │
                     │                                │
                     │  ┌─────────────────────────┐  │
                     │  │   Production EC2         │  │
                     │  │   ip: 35.75.189.198      │  │
                     │  │   (Elastic IP — fixed!)  │  │
                     │  │   Runner: My-portfolio-  │  │
                     │  │           production     │  │
                     │  │   Docker: nginx:alpine   │  │
                     │  │   Port: 80 + 443         │  │
                     │  │   SSL: Let's Encrypt     │  │
                     │  │   https://ec2.mdfahim.dev│  │
                     │  └─────────────────────────┘  │
                     └───────────────────────────────┘
                                    ↑
                     ┌──────────────┴──────────────┐
                     │         NAMECHEAP DNS        │
                     │                              │
                     │  mdfahim.dev → 145.79.14.205 │
                     │  (Hostinger — portfolio)     │
                     │                              │
                     │  ec2.mdfahim.dev →           │
                     │  35.75.189.198               │
                     │  (Production EC2)            │
                     └──────────────────────────────┘
```

---

## 🛠 Tech Stack

| Layer | Tool |
|---|---|
| Web server | nginx:alpine (Docker) |
| CI/CD | GitHub Actions |
| Container registry | Docker Hub |
| Cloud | AWS EC2 — ap-northeast-1 (Tokyo) |
| SSL | Let's Encrypt (Certbot) |
| DNS | Namecheap |
| Security scan | OWASP ZAP, Trivy, Gitleaks |
| Performance | Lighthouse CI |

---

## 📋 Workflows

| File | Trigger | Job |
|---|---|---|
| `docker-push.yml` | push: main, staging | Build & push Docker image |
| `deploy-staging.yml` | workflow_run (staging branch) | Auto deploy to staging EC2 |
| `deploy-production.yml` | workflow_dispatch (manual) | Deploy to production with approval |
| `secret-scan.yml` | push | Gitleaks secret leak check |
| `code-quality.yml` | push | HTMLHint code quality |
| `docker-lint.yml` | push | Hadolint Dockerfile check |
| `dependencies-scan.yml` | push | Trivy vulnerability scan |
| `owasp-zap.yml` | after Deploy Production | Live site security scan |
| `lighthouse.yml` | after Deploy Production | Performance & SEO score |
| `ssl-check.yml` | Every Monday 9AM UTC | SSL certificate expiry |
| `broken-link.yml` | Every Monday 9AM UTC | Broken link check |

---

## 🔄 Daily Workflow

```bash
# 1. Work on staging branch
git checkout staging
# make changes
git add . && git commit -m "description" && git push origin staging

# 2. Test at http://52.196.204.12

# 3. Merge to production
# GitHub → Pull requests → New PR → base: main ← compare: staging → Merge

# 4. Deploy to production
# Actions → Deploy Production → Run workflow → Approve
```

---

## 🏗 Infrastructure

| Resource | Details |
|---|---|
| Production EC2 | 35.75.189.198 (Elastic IP) |
| Staging EC2 | 52.196.204.12 |
| Region | ap-northeast-1 (Tokyo) |
| Docker Hub | fahim017803/mdfahim.dev |
| Production URL | https://ec2.mdfahim.dev |

---

## 👤 Author

**Md Mainul Islam Fahim** — DevOps-Focused IT Student | Sydney, Australia | Josh Batch 10