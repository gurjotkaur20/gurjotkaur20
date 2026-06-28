# AWS Networking, Kubernetes, and DevOps Q&A

## AWS Networking

### 1. When would you choose ALB vs NLB, and why?

| Feature | ALB | NLB |
|---------|-----|-----|
| **Layer** | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) |
| **Performance** | Good for web apps | Extreme performance, millions of requests/sec |
| **Use Case** | API Gateway, microservices, path-based routing | Gaming, IoT, real-time data, extreme throughput |
| **Latency** | ~100ms | <100μs |
| **Cost** | Lower | Higher |

**Choose ALB if:** Web applications, path/host-based routing, API Gateway

**Choose NLB if:** Ultra-low latency, extreme throughput, non-HTTP protocols (TCP/UDP)

---

### 2. Difference between: Security Groups vs NACLs, NAT Gateway vs NAT Instance

**Security Groups vs NACLs:**

| Aspect | Security Groups | NACLs |
|--------|-----------------|-------|
| **Layer** | Instance level | Subnet level |
| **State** | Stateful (allow inbound → auto-allow outbound) | Stateless (explicit rules for in/out) |
| **Rules** | Allow only | Allow/Deny |
| **Performance** | Flexible | More efficient |
| **Use** | Instance-specific rules | Network-wide rules |

**NAT Gateway vs NAT Instance:**

| Aspect | NAT Gateway | NAT Instance |
|--------|-------------|--------------|
| **Management** | AWS managed | Self-managed EC2 |
| **Availability** | Highly available (built-in redundancy) | Single point of failure |
| **Performance** | 5 Gbps baseline, 45 Gbps burst | Depends on instance type |
| **Cost** | Pay per hour + data | EC2 instance cost |
| **Setup** | Easy, one-click | Manual configuration |

**Choose NAT Gateway** for production. **Choose NAT Instance** for testing/cost reduction.

---

### 3. How do you implement private EKS cluster networking?

```yaml
# Key Components:
1. Private Subnets for worker nodes (no direct internet)
2. NAT Gateway in public subnet for outbound traffic
3. VPC Endpoints for AWS services (S3, ECR, CloudWatch)
4. Private API Server endpoint (enabled by default)
5. Bastion host for admin access

# Kubernetes Service Access:
- Internal ALB/NLB for pod-to-pod communication
- VPC Link for private API Gateway to ALB
- Security Groups restrict traffic to necessary ports
```

---

### 4. How do you troubleshoot: Instance not reachable? Pod cannot connect to database?

**Instance not reachable:**

```
1. Check Security Group inbound rules
2. Check NACL rules (allow/deny)
3. Verify Network Interface (ENI) attached
4. Check routing table for internet route
5. Ping from bastion/NAT Gateway
6. tcpdump for packet capture
7. VPC Flow Logs for traffic analysis
```

**Pod cannot connect to database:**

```
1. Verify pod → database security group rules
2. Check database subnet reachability
3. DNS resolution: kubectl exec -it pod -- nslookup db-service
4. Network policies blocking traffic (check NetworkPolicy)
5. Check kube-proxy and iptables rules
6. Port number correct in connection string
7. Database service endpoints healthy
```

---

### 5. How do you design secure cross-account VPC communication?

```yaml
# Option 1: VPC Peering
- Establish peering between VPCs
- Adjust routing tables
- Security groups allow cross-account traffic
- No data egress charges

# Option 2: Transit Gateway (Recommended)
- Centralized hub-and-spoke architecture
- Route tables control traffic
- Attach multiple VPCs/on-premises networks
- CloudWatch monitoring

# Option 3: PrivateLink
- Expose services via network interfaces
- Consumer accesses via VPC endpoint
- No IP space overlaps needed
- Secure and scalable

# Best Practice:
- Use least privilege security groups
- Enable VPC Flow Logs for audit
- Implement CloudWatch alarms
- Document routing policies
```

---

### 6. Explain IAM role assumption and STS

**IAM Role Assumption:**

```
1. Service A has IAM Role "RoleA" with permissions
2. Service B needs to use RoleA permissions
3. Service B assumes RoleA via STS AssumeRole API
4. Receives temporary credentials (AccessKey, SecretKey, SessionToken)
5. Uses credentials to access resources
6. Credentials expire after TTL (default 1 hour)

Benefits:
- No long-lived credentials
- Audit trail in CloudTrail
- Fine-grained permissions
- Cross-account access
```

**Example:**

```bash
# Service B assumes RoleA
aws sts assume-role --role-arn arn:aws:iam::ACCOUNT:role/RoleA \
                    --role-session-name session-name
# Returns temporary credentials used in subsequent API calls
```

---

### 7. How do you implement MFA and enforce it across accounts?

```yaml
# 1. Enable MFA for IAM users
- aws iam enable-mfa-device --user-name username

# 2. Force MFA for console login
- IAM policy: Deny all actions if not MFA'd
- Condition: "aws:MultiFactorAuthPresent": "true"

# 3. Cross-account MFA enforcement
- Central account: Require MFA in trust policy
- Member accounts: Reference MFA in assume-role conditions
- Service Control Policies (SCPs): Organization-wide MFA requirement

# 4. MFA for programmatic access
- Require MFA device for AssumeRole calls
- Session tokens include MFA validation timestamp
```

---

## Infrastructure as Code (Terraform)

### 8. How do you structure Terraform for multi-environment (dev, stage, prod)?

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── stage/
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── prod/
│       ├── terraform.tfvars
│       └── backend.tf
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── rds/
│   └── alb/
├── variables.tf
├── outputs.tf
└── main.tf

# Key Points:
- Each environment has separate state file
- Variables in terraform.tfvars for each env
- Shared modules reduce code duplication
- Backend stores state remotely per environment
```

---

### 9. How do you manage Terraform state securely?

```yaml
# Best Practices:
1. Remote State: Use S3 + DynamoDB for locking
2. Encryption: Enable S3 encryption and versioning
3. Access Control: IAM policies restrict who can access state
4. Isolation: Separate S3 buckets per environment
5. State Locking: DynamoDB prevents concurrent modifications
6. No Secrets in State: Use aws_secretsmanager_secret_version
7. Backup: Enable S3 versioning and MFA delete
8. Audit: CloudTrail logs all state file access

# Example Backend Config:
terraform {
  backend "s3" {
    bucket           = "my-terraform-state"
    key              = "prod/terraform.tfstate"
    region           = "us-east-1"
    encrypt          = true
    dynamodb_table   = "terraform-locks"
  }
}
```

---

### 10. How do you prevent configuration drift?

```yaml
# 1. Version Control
- All Terraform code in Git
- Code reviews before merge
- Branch protection rules

# 2. CI/CD Pipeline
- terraform plan before apply
- Automated testing
- Manual approval for prod

# 3. Continuous Monitoring
- terraform plan regularly (daily)
- Alert on differences
- Automated remediation or manual review

# 4. Policy as Code
- Sentinel or OPA policies enforce standards
- Prevent non-compliant resources

# 5. Resource Tagging
- Tag all resources
- Regularly audit untagged resources
- Cost tracking

# 6. Scheduled Drift Detection
- CloudFormation Drift Detection
- Custom scripts for Terraform
- Alerts on drift detection
```

---

### 11. What's the difference between: count vs for_each?

| Feature | count | for_each |
|---------|-------|----------|
| **Syntax** | `count.index` (numeric) | `each.key` / `each.value` (map) |
| **When to use** | Simple repetition (N copies) | Dynamic resources with unique identifiers |
| **Readability** | Less readable | More readable |
| **Drift** | Fragile (index shifts break references) | Stable (key-based) |
| **Performance** | Similar | Similar |

**Example:**

```hcl
# count - creates 3 subnets
resource "aws_subnet" "main" {
  count             = 3
  cidr_block        = "10.0.${count.index}.0/24"
}

# for_each - creates subnets for each env
resource "aws_subnet" "main" {
  for_each   = toset(["dev", "stage", "prod"])
  cidr_block = "10.0.${each.key}.0/24"
}
```

**Use for_each** in production for better maintainability.

---

### 12. What's the difference between: depends_on usage?

```hcl
# Implicit dependency (Terraform infers)
resource "aws_instance" "app" {
  subnet_id = aws_subnet.main.id  # Implicit dependency
}

# Explicit dependency (when Terraform can't infer)
resource "aws_iam_role_policy_attachment" "example" {
  role       = aws_iam_role.example.name
  policy_arn = aws_iam_policy.example.arn
  
  depends_on = [
    aws_iam_role.example,
    aws_iam_policy.example
  ]
}

# Use depends_on when:
- Side effects not captured in configuration
- Resource creation order matters but not explicit
- API dependencies not obvious
```

---

### 13. How do you manage remote state locking?

```yaml
# 1. DynamoDB Table for Locking
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    dynamodb_table = "terraform-locks"  # Required for locking
    region         = "us-east-1"
  }
}

# 2. DynamoDB Table Requirements
- Table name: terraform-locks (or custom)
- Primary key: LockID (string)
- Billing mode: Pay-per-request (or provisioned)
- TTL: Optional for auto-cleanup

# 3. Lock Behavior
- terraform plan: Acquires lock briefly
- terraform apply: Holds lock until complete
- terraform destroy: Holds lock until complete
- Timeout: Configurable (default: 0 = wait forever)

# 4. Release Stuck Locks
terraform force-unlock LOCK_ID
# Use cautiously - can cause conflicts
```

---

### 14. How do you modularize Terraform for enterprise use?

```
# Module Structure
modules/
├── vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
├── eks/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── rds/

# Root Module Usage
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr            = var.vpc_cidr
  availability_zones  = var.availability_zones
  environment         = var.environment
}

module "eks" {
  source = "./modules/eks"
  
  vpc_id            = module.vpc.vpc_id
  subnet_ids        = module.vpc.subnet_ids
  cluster_name      = "${var.environment}-cluster"
  
  depends_on = [module.vpc]
}

# Best Practices:
1. One responsibility per module
2. Clear inputs/outputs documented
3. Version modules in separate repos (registry)
4. Use semantic versioning
5. Test modules with terratest
```

---

## Kubernetes Deep Dive

### 15. How does pod-to-pod communication work across nodes?

```
Node 1                          Node 2
Pod A (10.244.1.2)              Pod B (10.244.2.3)
    │                               │
    ├─ eth0 (veth pair)             ├─ eth0 (veth pair)
    │                               │
    └─ docker0/cni0 Bridge      └─ docker0/cni0 Bridge
       (10.244.1.1)                (10.244.2.1)
          │                           │
          └─ Node Network Interface   │
             (10.0.1.0/24)            │
                 │                    │
                 └────────────────────┤
                  AWS VPC Subnet      │
                  Routing Table       │
                  Network ACLs        │
                  Security Groups     │
                       │              │
                       │              │
                       └──────────────┘

# Flow:
1. Pod A → docker0 bridge
2. Bridge checks ARP table for 10.244.2.3
3. Not local → queries routing table
4. Routing table: 10.244.0.0/16 → Node2 Interface
5. CNI plugin encapsulates packet (VXLAN/Direct Route)
6. Packet travels over VPC network
7. Arrives at Node2 interface → local routing
8. Decapsulation → docker0 bridge → Pod B
```

---

### 16. How would you: Debug a crashing pod?

```bash
# 1. Check pod status and events
kubectl describe pod <pod-name>
# Look for: CrashLoopBackOff, ImagePullBackOff, Pending

# 2. View logs (current + previous)
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # If crashed
kubectl logs <pod-name> -f          # Follow logs

# 3. Check resource limits
kubectl describe node <node-name>   # Node resources
kubectl top pod <pod-name>          # Pod CPU/Memory

# 4. Execute into pod (if running)
kubectl exec -it <pod-name> -- /bin/bash
# Check processes, disk space, network

# 5. Check readiness/liveness probes
kubectl get pod <pod-name> -o yaml | grep -A 5 "probe"

# 6. Events and debugging
kubectl get events --sort-by='.lastTimestamp'
kubectl debug pod <pod-name> -it --image=busybox
```

---

### 17. How would you: Debug high memory usage?

```bash
# 1. Identify memory hog
kubectl top pods --all-namespaces | sort --reverse -k 3 -h

# 2. Check limits vs requests
kubectl get pod <pod-name> -o yaml | grep -A 10 "resources:"

# 3. Memory leaks
kubectl logs <pod-name> | grep -i "memory\|oom"

# 4. Check container memory
kubectl exec -it <pod-name> -- ps aux
kubectl exec -it <pod-name> -- free -h

# 5. Java heap (if Java app)
kubectl exec -it <pod-name> -- jmap -heap <pid>

# 6. Monitor over time
kubectl top pod <pod-name> --containers

# 7. Increase limits (if needed)
# Edit deployment limits in spec.containers[].resources.limits.memory
```

---

### 18. Difference between: Deployment vs StatefulSet

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| **Identity** | Interchangeable replicas | Named, ordered identity (pod-0, pod-1, etc.) |
| **Persistence** | Ephemeral | Persistent volumes per pod |
| **Networking** | ClusterIP | Headless service (stable DNS) |
| **Scaling** | Parallel | Sequential (pod-0 → pod-1 → pod-2) |
| **Use Case** | Web servers, APIs, stateless apps | Databases, queues, ordered deployment |
| **Update Strategy** | Rolling update all | Rolling update ordered |

---

### 19. ClusterIP vs NodePort vs LoadBalancer

See detailed comparison in [Kubernetes Services](./k8s_services.md)

**Quick Summary:**
- **ClusterIP**: Internal communication only (default)
- **NodePort**: Expose on every node port (30000-32767)
- **LoadBalancer**: Cloud provider load balancer (ALB/NLB)

---

### 20. How do you implement: Pod security?

```yaml
# Pod Security Policy (deprecated in 1.25+, use Pod Security Standards)
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
  - ALL
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'MustRunAs'
  fsGroup:
    rule: 'MustRunAs'
  readOnlyRootFilesystem: true

# Pod Security Standards (preferred)
apiVersion: v1
kind: Pod
metadata:
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
```

---

### 21. How do you implement: RBAC?

```yaml
# 1. Create Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# 2. Create RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: User
  name: "user@example.com"
  apiGroup: rbac.authorization.k8s.io

# 3. ClusterRole for cluster-wide permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

---

### 22. How do you implement: Network policies?

```yaml
# Deny all ingress by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress

# Allow specific traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

---

### 23. How do you scale workloads efficiently?

```yaml
# 1. Horizontal Pod Autoscaling (HPA)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

# 2. Vertical Pod Autoscaling (VPA)
# Automatically adjusts CPU/memory requests

# 3. Cluster Autoscaler
# Adds nodes when pods can't be scheduled

# 4. Custom Metrics Scaling
# Scale based on application-specific metrics

# Best Practices:
- Set resource requests/limits
- Use multiple metrics (CPU, memory, custom)
- Configure cooldown periods
- Monitor HPA events
```

---

## Troubleshooting Scenarios

### 24. Scenario 1: Production is down. ALB is healthy, but users cannot access the application. How do you troubleshoot?

```bash
# 1. Check ALB target health
aws elbv2 describe-target-health --target-group-arn <arn>

# 2. Check ALB listener rules
aws elbv2 describe-listeners --load-balancer-arn <arn>

# 3. Check security groups
aws ec2 describe-security-groups --group-ids <sg-id>

# 4. Check network ACLs
aws ec2 describe-network-acls --filters "Name=association.subnet-id,Values=<subnet-id>"

# 5. Test connectivity
curl -v http://<alb-dns>

# 6. Check application logs
ssh <instance> && tail -f /var/log/app.log

# 7. Check DNS
nslookup <alb-dns>

# 8. Check routing table
aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=<subnet-id>"

# Most Common Issues:
- Target security group blocks traffic
- Application not listening
- Wrong port in listener rules
- Health check failing (app not ready)
- NACL blocking traffic
```

---

### 25. Scenario 2: Terraform apply failed midway. Some resources created, some not. What do you do?

```bash
# 1. Check state file
terraform show | head -50

# 2. Identify the failure
# Check: Resource limits, API throttling, IAM permissions

# 3. Retry apply (safest)
terraform apply

# 4. If retry fails - fix and re-apply
terraform taint aws_instance.example
terraform apply

# 5. Recovery Steps:
# a) Review error logs
# b) Fix root cause (IAM, limits, etc.)
# c) terraform plan (verify no changes)
# d) terraform apply
# e) Verify all resources
# f) Commit to Git

# Prevention:
- Use -parallelism=1 for sequential apply
- Set higher retry counts
- Implement pre-apply checks
- Test in non-prod first
```

---

### 26. Scenario 3: Kubernetes pods are restarting intermittently in production. How do you investigate?

```bash
# 1. Check pod restart count
kubectl get pods -A

# 2. View logs
kubectl logs <pod-name> --tail=100
kubectl logs <pod-name> --previous

# 3. Check events
kubectl describe pod <pod-name>

# 4. Monitor resource usage
kubectl top pod <pod-name>
kubectl top node

# 5. Check health probes
kubectl get pod <pod-name> -o yaml | grep -A 10 "probe"

# 6. Check node conditions
kubectl describe node <node-name>

# 7. Check kernel logs
journalctl -u kubelet -n 100

# Common Causes:
- Memory leak → OOMKilled
- Failing liveness probe → restarts
- Resource limits too low
- Application errors

# Fix:
- Increase resource limits
- Fix application bug
- Adjust probe thresholds
```

---

### 27. Scenario 4: Cloud cost suddenly increased by 40%. How do you analyze?

```bash
# 1. Check AWS Billing
aws ce get-cost-and-usage --time-period Start=2024-01-01,End=2024-01-31 --granularity MONTHLY --metrics "BlendedCost"

# 2. Identify biggest cost drivers
aws ce get-cost-and-usage --time-period Start=2024-01-01,End=2024-01-31 --granularity DAILY --group-by Type=DIMENSION,Key=SERVICE --metrics "UnblendedCost"

# 3. Check for new resources
aws resourcegroupstaggingapi get-resources | grep -i cost

# 4. Check for unused resources
aws ec2 describe-volumes --filters "Name=status,Values=available"

# 5. Check for high data transfer
# Review CloudFront, NAT Gateway, VPC peering

# 6. Check database costs
aws rds describe-db-instances

# Common Causes:
- New resources provisioned
- Data transfer costs (NAT Gateway)
- RDS multi-AZ enabled
- Reserved instances expired
- Unused EBS/Snapshots

# Prevention:
- Tag all resources
- Cost Anomaly Detection
- Reserved Instances for steady workloads
- Savings Plans
- Regular cost reviews
```

---

## Additional Resources

- [Kubernetes Services Deep Dive](./k8s_services.md)
- [Kubernetes Core Components](./k8s_core.md)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [Terraform Documentation](https://www.terraform.io/docs/)
