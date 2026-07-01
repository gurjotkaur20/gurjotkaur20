# CI/CD Fundamentals

## What is CI/CD?

**CI/CD** is an automated software delivery pipeline that continuously builds, tests, validates, and deploys applications with minimal manual intervention.

**Two Major Phases:**
- **CI (Continuous Integration):** Validate every code change
- **CD (Continuous Delivery/Deployment):** Release validated changes to environments

```
Developer → Git Push → CI Pipeline (Build, Test, Validate) → CD Pipeline (Deploy)
```

---

## High-Level Architecture

```
                    Developer
                        │
                  Git Push / PR
                        │
                        ▼
            Source Control (GitHub/GitLab)
                        │
                        ▼
                CI Pipeline Triggered
                 │      │      │
              Build   Tests   Lint
                 │      │      │
                 └──────┴──────┘
                        │
                        ▼
                Docker Build & Push
                        │
                        ▼
                CD Pipeline Triggered
                        │
            ┌───────────┼───────────┐
            │           │           │
          Dev         Staging      Prod
            │           │           │
         Tests       Tests      Approval
            │           │           │
            └───────────┴───────────┘
                        │
                        ▼
              Health Checks & Monitoring
```

---

## Continuous Integration (CI)

**Purpose:** Validate every code change before merge/release.

**Typical CI Steps:**

```
Developer Push
      │
Checkout Code
      │
Install Dependencies
      │
Run Unit Tests
      │
Static Analysis (Lint, SAST)
      │
Security Scans (Secrets, Dependencies)
      │
Build Application
      │
Create Docker Image
      │
Push Artifact to Registry
      │
✅ CI Success or ❌ Failure
```

**If any step fails → Pipeline stops (fail fast)**

---

## Continuous Delivery vs Continuous Deployment

### Continuous Delivery (CD)

- Every successful build is **deployable**
- Production deployment requires **manual approval**
- Common in enterprises (compliance, risk management)

```
CI → QA → Approval → Production
```

### Continuous Deployment

- Every successful build is **automatically deployed** to production
- No manual approval needed
- Requires high test coverage and confidence

```
CI → Production (automatic)
```

**Most enterprises prefer Continuous Delivery for production.**

---

## CI/CD Pipeline Stages

### 1. Source

**Trigger:** Push, Pull Request, or Merge

Webhook automatically initiates pipeline.

### 2. Build

Compile and package application.

```bash
mvn package      # Java
go build         # Go
npm run build    # Node.js
cargo build      # Rust
```

### 3. Test

```
Unit Tests
Integration Tests
API Tests
UI Tests
```

Run in parallel when possible.

### 4. Static Analysis

Analyze code quality and bugs.

**Tools:**
- SonarQube
- ESLint
- golangci-lint

**Checks:**
- Code smells
- Complexity
- Test coverage
- Bugs

#### SonarQube: Code Quality Platform

**What is SonarQube?**

SonarQube is a static code analysis (SAST) platform that automatically analyzes source code for bugs, vulnerabilities, code smells, code duplication, and test coverage.

**How SonarQube Works:**

```
Developer Push
      │
CI/CD Pipeline Triggered
      │
Sonar Scanner Runs
      │
SonarQube Server Analyzes
      │
Analysis Report Generated
      │
Quality Gate Evaluation
      │
✅ Pass or ❌ Fail
```

**What SonarQube Checks:**

```
✅ Bugs – Potential coding errors
✅ Vulnerabilities – Security issues (SAST)
✅ Code Smells – Maintainability issues
✅ Code Coverage – Test coverage percentage
✅ Code Duplication – Repeated code blocks
✅ Quality Gate – Pass/fail criteria
```

**Example Issues Detected:**

```javascript
// ❌ Hardcoded credential (vulnerability)
if (password == "admin123") {
    // ...
}

// ❌ Weak comparison (== instead of ===)
// ❌ Use const instead of var
var x = 10;

// ❌ High cognitive complexity
function calculate(a, b) {
    // 300+ lines of nested logic
}
```

**Quality Gates:**

Define conditions that must pass before merge:

```yaml
No Critical Vulnerabilities
No Blocker Bugs
Test Coverage ≥ 80%
Code Duplication < 3%
```

If Quality Gate fails:

```
Build Fails → Code cannot be merged
```

**Supported Languages:**

```
Java, JavaScript, TypeScript, Python, Go
C/C++, C#, Kotlin, PHP, Ruby, and others
```

**Integration in CI/CD:**

```yaml
# Example: GitHub Actions
- name: Run SonarQube Scan
  run: |
    sonar-scanner \
      -Dsonar.projectKey=myapp \
      -Dsonar.sources=. \
      -Dsonar.host.url=https://sonarqube.example.com
```

### 5. Security Scanning

#### SAST (Static Application Security Testing)
```
Semgrep
Checkmarx
SonarQube
```

#### Dependency Scanning
```
Snyk
OWASP Dependency Check
npm audit
```

#### Secret Scanning
```
Gitleaks
TruffleHog
```

#### Container Scanning
```
Trivy
Clair
Anchore
```

#### Infrastructure Scanning
```
Checkov
tfsec
```

### 6. Package

Create deployable artifact.

```
Docker Image
JAR/WAR
Binary
Helm Chart
```

### 7. Artifact Repository

Store versioned artifacts.

```
ECR (AWS)
Docker Hub
Artifactory
Nexus
```

### 8. Deploy

Deploy to environments.

```
Development
Staging/UAT
Production
```

**Deployment Tools:**
- Helm
- Argo CD
- Flux
- Terraform
- Kustomize

### 9. Verification

Post-deployment checks.

```
Smoke Tests
Health Checks
Readiness Probes
API Tests
```

### 10. Monitoring

Continuous observation.

```
Prometheus (Metrics)
Grafana (Dashboards)
Loki (Logs)
Tempo (Traces)
Alerting (PagerDuty, Slack)
```

---

## CI/CD in Kubernetes

```
Git Repository
      │
      ▼
CI Pipeline (Build Docker Image)
      │
      ▼
Container Registry (ECR, Docker Hub)
      │
      ▼
GitOps Repository (Updated manifests)
      │
      ▼
Argo CD (Pulls from Git)
      │
      ▼
Kubernetes Cluster
      │
      ▼
Running Pods
```

**Key Separation:**
- **CI** creates the artifact (Docker image)
- **CD** deploys the artifact (via Argo CD)

---

## Deployment Strategies

### Rolling Update (Kubernetes Default)

```
Old Old Old   →   New Old Old   →   New New Old   →   New New New
```

**Pros:** Minimal downtime, gradual rollout

**Cons:** Complex rollback during update

### Blue-Green Deployment

```
Blue (Current) → Green (New) → Switch Traffic → Rollback Quick
```

**Pros:** Instant rollback by switching traffic

**Cons:** Requires double resources temporarily

### Canary Deployment

```
10% New Users   →   25% New Users   →   50% New Users   →   100% New
```

**Pros:** Detect issues on small user subset

**Cons:** Complex traffic routing, longer rollout

### Recreate

```
Stop Old   →   Start New
```

**Pros:** Simple implementation

**Cons:** Downtime during deployment

---

## CI/CD Components

| Component | Purpose | Examples |
|-----------|---------|----------|
| **Source Control** | Version management | GitHub, GitLab, Bitbucket |
| **CI Server** | Orchestrate pipeline | Jenkins, GitHub Actions, GitLab CI, CircleCI |
| **Build Tool** | Compile code | Maven, Gradle, npm, Go |
| **Artifact Repository** | Store binaries | ECR, Docker Hub, Artifactory, Nexus |
| **Container Registry** | Store images | ECR, Docker Hub, Quay |
| **Deployment Tool** | Deploy to K8s | Argo CD, Flux, Helm |
| **Monitoring** | Observe health | Prometheus, Grafana, DataDog |
| **Logging** | Track events | Loki, ELK, Splunk |

---

## Best Practices

✅ **Keep pipelines fast and deterministic**
   - Aim for < 10 minutes per pipeline run
   - Parallelize tests where possible

✅ **Fail fast on errors**
   - Stop pipeline immediately on failure
   - Don't waste resources on broken builds

✅ **Use immutable versioning**
   - Tag artifacts with Git SHA or semantic versions
   - Never rely on `latest` tag

✅ **Manage secrets securely**
   - Use secrets managers (Vault, AWS Secrets Manager)
   - Never hardcode credentials in source control

✅ **Promote same artifact through environments**
   - Build once, deploy to dev → staging → prod
   - Don't rebuild for each environment

✅ **Require approvals for production**
   - Manual approval before production deployment
   - Audit trail for compliance

✅ **Ensure pipeline idempotency**
   - Running twice should produce same result
   - Enable safe retries

✅ **Run tests in parallel**
   - Distribute tests across agents
   - Reduce overall pipeline time

✅ **Implement comprehensive testing**
   - Unit, integration, API, UI tests
   - Aim for > 80% code coverage

✅ **Monitor pipeline health**
   - Track build success rates
   - Alert on pipeline failures

---
