# mdfahim.dev — DevSecOps Portfolio Pipeline

> Personal portfolio site built with a production-grade DevSecOps pipeline featuring multi-environment deployment, automated security scanning, and manual production approval gates.
>
> **Live site:** [https://ec2.mdfahim.dev](https://ec2.mdfahim.dev) | **Portfolio:** [https://mdfahim.dev](https://mdfahim.dev)

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

| Layer | Tool | Purpose |
|---|---|---|
| Web server | nginx:alpine (Docker) | Serve static HTML, SSL termination |
| Containerisation | Docker + Docker Compose | Reproducible builds and deployments |
| CI/CD | GitHub Actions | Automated build, test, deploy pipeline |
| Container registry | Docker Hub | Store and version Docker images |
| Cloud | AWS EC2 ap-northeast-1 (Tokyo) | Host staging and production servers |
| Static IP | AWS Elastic IP | Prevent IP change on EC2 restart |
| SSL/TLS | Let's Encrypt (Certbot) | Free auto-renewing HTTPS certificate |
| DNS | Namecheap Advanced DNS | Domain → IP mapping |
| Security scanning | OWASP ZAP | Live site vulnerability scan |
| Dependency scan | Trivy | Docker image CVE scanning |
| Secret detection | Gitleaks | Prevent secrets from being committed |
| Code quality | HTMLHint | HTML linting and validation |
| Dockerfile lint | Hadolint | Dockerfile best practices |
| Performance | Lighthouse CI | Performance, SEO, Accessibility scores |

---

## 📋 Workflows

| File | Trigger | Runner | Job |
|---|---|---|---|
| `docker-push.yml` | push: main, staging | ubuntu-latest | Build Docker image and push to Docker Hub |
| `deploy-staging.yml` | workflow_run: docker build (staging) | Staging EC2 (self-hosted) | Auto deploy to staging server |
| `deploy-production.yml` | workflow_dispatch (manual) | Production EC2 (self-hosted) | Deploy to production with manual approval gate |
| `secret-scan.yml` | push: main, staging | ubuntu-latest | Gitleaks — detect hardcoded secrets |
| `code-quality.yml` | push: main, staging | ubuntu-latest | HTMLHint — validate HTML structure |
| `docker-lint.yml` | push: main, staging | ubuntu-latest | Hadolint — Dockerfile best practices |
| `dependencies-scan.yml` | push: main, staging | ubuntu-latest | Trivy — scan image for known CVEs |
| `owasp-zap.yml` | workflow_run: Deploy Production | ubuntu-latest | OWASP ZAP — live site security scan |
| `lighthouse.yml` | workflow_run: Deploy Production | ubuntu-latest | Lighthouse CI — performance and SEO |
| `ssl-check.yml` | schedule: Mon 9AM UTC | ubuntu-latest | Alert if SSL certificate expires soon |
| `broken-link.yml` | schedule: Mon 9AM UTC | ubuntu-latest | Scan site for broken links |

---

## 🔒 Security Layers

```
Code Level
  ├── Gitleaks        → detect secrets before they reach GitHub
  ├── HTMLHint        → enforce HTML quality standards
  ├── Hadolint        → enforce Dockerfile security best practices
  └── Trivy           → scan Docker image for known CVEs

Network Level
  ├── HTTPS (443)     → TLS 1.2 / 1.3 via Let's Encrypt
  ├── HTTP → HTTPS    → nginx permanent redirect (301)
  └── Security headers via nginx:
        X-Frame-Options: DENY
        X-Content-Type-Options: nosniff
        Referrer-Policy: strict-origin-when-cross-origin
        Strict-Transport-Security: max-age=31536000

Runtime Level
  └── OWASP ZAP       → automated live site penetration scan

Monitoring
  ├── SSL expiry check → every Monday
  └── Broken link scan → every Monday
```

---

## 🏗 Infrastructure Details

| Resource | Value |
|---|---|
| Production EC2 IP | 35.75.189.198 (Elastic IP — never changes) |
| Staging EC2 IP | 52.196.204.12 |
| Region | ap-northeast-1 (Tokyo) |
| AMI | Ubuntu 24.04 LTS |
| Docker | Official install script (Compose v2 plugin) |
| Production runner | My-portfolio-production (labels: self-hosted, Linux, X64, production) |
| Staging runner | My-porfolio-staging (labels: self-hosted, Linux, X64) |
| Docker Hub repo | fahim017803/mdfahim.dev |
| Production URL | https://ec2.mdfahim.dev |
| Staging URL | http://52.196.204.12 |

---

## 🔄 Daily Workflow

```bash
# 1. Work on staging branch
git checkout staging

# 2. Make changes to index.html or config files

# 3. Push to staging — pipeline runs automatically
git add .
git commit -m "feat: description of change"
git push origin staging

# 4. Wait for GitHub Actions to complete:
#    Secret Scan ✓ → Code Quality ✓ → Docker Lint ✓ → Dep Scan ✓
#    → Docker Build & Push ✓ → Deploy Staging ✓

# 5. Test at http://52.196.204.12

# 6. Create Pull Request: staging → main
#    GitHub → Pull requests → New pull request → Merge

# 7. Deploy to production manually
#    Actions → Deploy Production → Run workflow → Approve
```

---

## 📁 Project Structure

```
mdfahim.dev/
├── index.html                          # Portfolio site
├── dockerfile                          # FROM nginx:alpine, EXPOSE 80
├── docker-compose.yml                  # Production: ports 80+443, SSL volumes
├── docker-compose.staging.yml          # Staging: port 80 only, no SSL
├── nginx.conf                          # Production: HTTPS + security headers
├── nginx.staging.conf                  # Staging: HTTP only
└── .github/
    └── workflows/
        ├── docker-push.yml
        ├── deploy-staging.yml
        ├── deploy-production.yml
        ├── secret-scan.yml
        ├── code-quality.yml
        ├── docker-lint.yml
        ├── dependencies-scan.yml
        ├── owasp-zap.yml
        ├── lighthouse.yml
        ├── ssl-check.yml
        └── broken-link.yml
```

---

## 📊 Lighthouse Scores

| Category | Score |
|---|---|
| Performance | 86 |
| Accessibility | 100 ✅ |
| Best Practices | 96 |
| SEO | 100 ✅ |

---

## 🧠 Key Lessons Learned

| Lesson | Detail |
|---|---|
| Config files first | Always verify docker-compose.yml before debugging elsewhere |
| Runner labels matter | Workflow `runs-on` labels must exactly match runner labels |
| `EXPOSE` vs port mapping | Only `EXPOSE 80` in Dockerfile — port mapping goes in docker-compose |
| letsencrypt permissions | Run `chown -R ubuntu:ubuntu /etc/letsencrypt/` after certbot |
| workflow_run reads main | Workflow files used by `workflow_run` trigger are always read from the default branch |
| Runner as service | Always use `svc.sh` — never run `./run.sh` manually alongside the service |
| Staging ≠ Production | Separate docker-compose files for staging and production |
| Elastic IP | Prevents IP change on EC2 restart — required for stable DNS and SSL |

---

## 👤 Author

**Md Mainul Islam Fahim** — DevOps-Focused IT Student | Sydney, Australia | Josh Batch 10
