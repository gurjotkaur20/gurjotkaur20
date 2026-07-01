# Argo CD Basics: GitOps for Kubernetes

## What is Argo CD?

**Argo CD** is a declarative, GitOps-based Continuous Delivery (CD) tool for Kubernetes.

**Core Principle:** Git is the single source of truth. Argo CD continuously ensures your Kubernetes cluster matches your Git repository.

**Traditional Deployment:**
```bash
Developer → kubectl apply -f deployment.yaml → Kubernetes
```

**Argo CD Deployment:**
```
Developer → git commit & push → Git Repository → Argo CD → Kubernetes
```

---

## Why Do We Need Argo CD?

### Problems with Manual Deployments

```
❌ Manual kubectl apply commands
❌ No audit trail of who deployed what
❌ Configuration drift (cluster differs from source)
❌ Different environments become inconsistent
❌ Difficult and error-prone rollbacks
❌ Hard to reproduce production state
```

### Advantages of Argo CD (GitOps)

```
✅ Git is the source of truth
✅ Full audit trail (Git commits)
✅ Automatic synchronization
✅ Version control for deployments
✅ Easy rollbacks (revert Git commit)
✅ Consistent across environments
✅ Infrastructure as Code (IaC)
```

---

## GitOps Principle

**Git Repository contains:**
```yaml
deployment.yaml
├─ replicas: 3
├─ image: myapp:1.0
└─ resources: {...}

service.yaml
ingress.yaml
configmap.yaml
```

**Developer Makes Change:**
```yaml
# deployment.yaml
replicas: 3  →  5
```

**Developer Commits:**
```bash
git commit -m "Scale to 5 replicas"
git push
```

**Argo CD Detects Change:**
```
Git: replicas = 5
Kubernetes: replicas = 3

Difference detected!
↓
Apply changes
↓
Kubernetes: replicas = 5 (synced)
```

---

## High-Level Architecture

```
                    Git Repository
                    (deployment.yaml)
                          │
                 (Poll every 3 min OR Webhook)
                          │
                          ▼
                   Argo CD Server
                    (Web UI, API)
                          │
        ┌─────────────────┴──────────────────┐
        │                                    │
        ▼                                    ▼
Repository Server              Application Controller
(Render manifests)             (Compare & Sync)
        │                                    │
        ├─ Clone Git repo                   ├─ Watch Git
        ├─ Read YAML/Helm/Kustomize        ├─ Watch Kubernetes
        └─ Convert to manifests             ├─ Detect drift
                                            ├─ Compare states
                                            └─ Apply changes
                                                   │
                                                   ▼
                                        Kubernetes API Server
                                                   │
                                                   ▼
                                          Kubernetes Cluster
```

---

## Argo CD Components

### 1. API Server

**Responsibilities:**
- Exposes REST API for external integrations
- Serves Web UI dashboard
- Handles CLI communication
- Authentication & authorization
- Application management API

**Used by:**
- Developers (Web UI)
- CI/CD pipelines (API calls)
- argocd CLI

---

### 2. Repository Server

**Responsibilities:**
- Clones Git repositories
- Reads and validates Kubernetes manifests
- Processes Helm charts (renders values)
- Processes Kustomize templates
- Converts everything to plain YAML manifests

**Supports:**
```
✅ Plain YAML
✅ Helm charts
✅ Kustomize overlays
✅ Jsonnet
✅ Custom plugins
```

**Example:**
```yaml
# Git repo contains
Chart.yaml
values.yaml
templates/deployment.yaml

↓ Repository Server processes ↓

deployment.yaml (rendered manifest)
service.yaml
```

---

### 3. Application Controller

**The Brain of Argo CD**

**Responsibilities:**
- Watches Git repository for changes
- Watches Kubernetes cluster state
- Compares desired state (Git) vs actual state (Kubernetes)
- Detects configuration drift
- Synchronizes cluster to match Git
- Generates sync status

**Key Operations:**
```
1. Poll Git (every 3 minutes by default)
2. Poll Kubernetes (continuous)
3. Compare both states
4. If different → Sync
5. If same → No action
```

**Sync Strategies:**
```
Auto Sync: Automatically apply Git changes
Manual Sync: Require human approval
```

---

### 4. Redis

**Purpose:**
- Caching layer
- Stores temporary data
- Improves performance
- Reduces Git API calls

---

## Argo CD Workflow

### Step 1: Application Definition

Create Git repository with Kubernetes manifests:

```bash
git clone https://github.com/user/app-config.git
cd app-config

# Create manifests
cat > deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
EOF

git add deployment.yaml
git commit -m "Add deployment"
git push
```

### Step 2: Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 3: Create Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/app-config.git
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Step 4: Argo CD Syncs Cluster

```
Argo CD detects Application CR
      ↓
Connects to Git repo
      ↓
Reads deployment.yaml (3 replicas)
      ↓
Checks Kubernetes cluster
      ↓
Creates 3 replicas
      ↓
Application SYNCED ✅
```

### Step 5: Developer Makes Change

```yaml
# deployment.yaml
replicas: 3 → 5

git commit -m "Scale to 5"
git push
```

### Step 6: Argo CD Detects & Syncs

```
Argo CD polls Git
      ↓
Detects replicas changed to 5
      ↓
Compares with Kubernetes (currently 3)
      ↓
SYNC OUT OF SYNC
      ↓
Auto Sync enabled → applies changes
      ↓
Scales to 5 replicas
      ↓
Application SYNCED ✅
```

---

## Sync Status

| Status | Meaning |
|--------|---------|
| **Synced** | Git state = Cluster state |
| **OutOfSync** | Git state ≠ Cluster state (drift detected) |
| **Unknown** | Cannot compare states |
| **Progressing** | Sync in progress |
| **Degraded** | Application health check failed |
| **Missing** | Application CRD not found |

---

## Sync Policies

### Automated Sync

```yaml
syncPolicy:
  automated:
    prune: true      # Remove resources not in Git
    selfHeal: true   # Auto-sync if drift detected
```

**Behavior:**
- Git change detected → Automatically apply
- Cluster drift detected → Automatically fix
- No manual approval needed

### Manual Sync

```yaml
syncPolicy: {}  # No automated sync
```

**Behavior:**
- Git changes are detected but NOT applied
- Requires manual approval (via UI or CLI)
- Safer for production

---

## Rollback

**Traditional Method:**
```bash
kubectl rollout undo deployment/myapp
```

**Argo CD Method:**
```bash
# Revert Git commit
git revert <commit-hash>
git push

# Argo CD automatically syncs to previous state
```

**Advantage:** Full audit trail in Git history.

---

## Key Benefits

✅ **Git as Source of Truth:** Single source of truth for all deployments

✅ **Audit Trail:** Every change tracked in Git commits

✅ **Automatic Synchronization:** Cluster stays in sync with Git

✅ **Drift Detection:** Immediately detects when cluster differs from Git

✅ **Easy Rollbacks:** Revert Git commits to rollback deployments

✅ **Multi-Environment:** Manage dev, staging, prod from same Git repo

✅ **Visibility:** Web UI shows all deployments and health

✅ **Scalability:** Manage multiple clusters with one Argo CD

✅ **Security:** No need to expose cluster to CI/CD tools (pull-based)

---

## Architecture Comparison

### Pull-based (Argo CD)

```
Kubernetes Cluster
      ↑
      └─ Pull from Git (secure, cluster initiates)
           ↑
         Git Repository
```

**Advantages:**
- Cluster pulls changes (more secure)
- No need for cluster credentials in CI/CD
- Automatic synchronization

### Push-based (Traditional CI/CD)

```
CI/CD Pipeline
      ↓
      → Push to Kubernetes Cluster (credentials needed)
           ↓
         Kubernetes Cluster
```

**Disadvantages:**
- Cluster credentials exposed to CI/CD
- Manual deployments required
- No automatic drift detection

---

## Common Use Cases

```
✅ Multi-cluster deployments
✅ GitOps workflows
✅ Environment promotion (dev → staging → prod)
✅ Helm releases management
✅ Kustomize-based deployments
✅ Progressive delivery with Argo Rollouts
✅ Infrastructure as Code (IaC)
```

---

## Getting Started Checklist

- [ ] Install Argo CD in Kubernetes cluster
- [ ] Create Git repository with manifests
- [ ] Configure Git credentials in Argo CD
- [ ] Create Argo CD Application
- [ ] Verify sync status in Web UI
- [ ] Make Git commit and verify auto-sync
- [ ] Test rollback by reverting commit

---

## Resources

- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Argo CD GitHub](https://github.com/argoproj/argo-cd)
- [GitOps Principles](https://opengitops.dev/)
- [Kubernetes Manifest Best Practices](https://kubernetes.io/docs/)
