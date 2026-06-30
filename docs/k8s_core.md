# Kubernetes Core

## What is Kubernetes?

**Kubernetes** is an open-source container management platform that:

- **Orchestrates containerized apps**: Deploys, manages, and scales them across a cluster
- **Originated at Google (2014)**: Now open-sourced and maintained by the CNCF (Cloud Native Computing Foundation)
- **Groups containers into pods**: Manages their full lifecycle from creation to termination
- **Maintains desired state**: Auto-restarts, replaces failed containers, and reschedules workloads on healthy nodes
- **Provides abstractions**: Services, deployments, and other primitives for building resilient distributed systems

This document covers essential Kubernetes concepts for administrators, DevOps engineers, and Site Reliability Engineers.

## Table of Contents

- [Cluster Architecture](#cluster-architecture)
  - [Control Plane Components](#control-plane-components)
    - [API Server](#api-server)
    - [Scheduler](#scheduler)
    - [Controller Manager](#controller-manager)
    - [ETCD](#etcd)
    - [Components Interaction](#components-interaction)
  - [Worker Node Components](#worker-node-components)
    - [Kubelet](#kubelet)
    - [Kube-proxy](#kube-proxy)
    - [Container Runtime](#container-runtime)
- [Networking](#networking)
  - [CNI (Container Network Interface)](#cni-container-network-interface)
  - [Pod-to-Pod Communication](#pod-to-pod-communication)
    - [Same Node Communication](#same-node-communication)
    - [Cross-Node Communication](#cross-node-communication)
  - [Services](#services)
    - [ClusterIP](#clusterip)
    - [NodePort](#nodeport)
    - [LoadBalancer](#loadbalancer)
    - [Ingress](#ingress)
  - [CNI Plugins](#cni-plugins)
    - [Calico](#calico)
    - [Flannel](#flannel)
    - [Cilium](#cilium)
  - [DNS Resolution](#dns-resolution)
- [ETCD & Cluster State](#etcd--cluster-state)
  - [What is ETCD](#what-is-etcd)
  - [Data Stored in ETCD](#data-stored-in-etcd)
  - [ETCD Backup](#etcd-backup)
  - [ETCD Restore](#etcd-restore)
- [Pods, Deployments, ReplicaSets](#pods-deployments-replicasets)
  - [Desired State vs Actual State](#desired-state-vs-actual-state)
  - [RollingUpdate Strategy](#rollingupdate-strategy)
  - [Container Crash Handling](#container-crash-handling)
- [Scheduling](#scheduling)
  - [Node Selection](#node-selection)
    - [nodeSelector](#nodeselector)
    - [nodeAffinity](#nodeaffinity)
  - [Taints and Tolerations](#taints-and-tolerations)
  - [Pod Priority and Preemption](#pod-priority-and-preemption)
  - [Workload Types](#workload-types)
    - [DaemonSet vs Deployment](#daemonset-vs-deployment)

---

## Cluster Architecture

### Control Plane (Cluster Management)

The control plane manages the cluster and makes global decisions about the cluster (scheduling, scaling, etc.).


![K8s architecture](image.png)

### Master Node (Control Plane) Overview

The **Master Node** (also called the **Control Plane**) is the brain of the Kubernetes cluster. It is responsible for managing whole cluster. It monitors the health check of all nodes in the cluster. It stores the members information regarding the different nodes, planning which containers are scheduled to which worker nodes, monitoring the containers and nodes, etc. So, when a worker node failed, it moves the workload from failed node to another healthy worker node. Kubernetes master is responsible for scheduling, provisioning, configuring and exposing API’s to the client. So, all these done by a master node using the components called as control plane components. Kubernetes takes care of service discovery, scaling, load balancing, self healing, leader election, etc. therefore, developers no longer have to build these services inside of their application.

```
┌─────────────────────────────────────────┐
│         Master Node / Control Plane     │
├─────────────────────────────────────────┤
│ • API Server (kube-apiserver)          │
│   → REST API endpoint, entry point     │
│ • Scheduler (kube-scheduler)           │
│   → Pod placement decisions            │
│ • Controller Manager (kube-controller) │
│   → Reconciliation loop, resource mgmt │
│ • ETCD (Data Store)                    │
│   → Persistent cluster state storage   │
└─────────────────────────────────────────┘
```

#### API Server

The **API Server** (`kube-apiserver`) is the central management entity and the **only component that directly communicates with ETCD**.

**Key Functions:**
- **REST API Gateway**: Exposes Kubernetes API and handles all REST operations
- **Authentication & Authorization**: Validates user credentials and permissions
- **Admission Control**: Validates and potentially modifies requests
- **ETCD Interface**: Only component that reads/writes to ETCD
- **Cluster Gateway**: Entry point for all administrative tasks

**How it Works:**
1. Receives API requests (kubectl, other components)
2. Authenticates and authorizes requests
3. Validates request format and content
4. Writes/reads data to/from ETCD
5. Returns response to client

**Example Interaction:**
```bash
kubectl create deployment nginx --image=nginx
# 1. kubectl → API Server (authentication)
# 2. API Server → ETCD (store deployment spec)
# 3. Controller Manager detects new deployment
# 4. Scheduler assigns pods to nodes
```

#### Scheduler

The **Scheduler** (`kube-scheduler`) determines **which node** a pod should run on based on resource requirements and constraints.

**Key Functions:**
- **Node Selection**: Chooses the best node for each pod
- **Resource Awareness**: Considers CPU, memory, storage requirements
- **Constraint Handling**: Respects nodeSelector, affinity, taints/tolerations
- **Load Balancing**: Distributes workloads across nodes

**Scheduling Process:**
1. **Filtering Phase**: Eliminates unsuitable nodes
   - Resource constraints (CPU/memory)
   - Node selectors and affinity rules
   - Taints and tolerations
   - Volume constraints

2. **Scoring Phase**: Ranks suitable nodes
   - Resource utilization balance
   - Spreading pods across zones
   - Preferred affinity rules

3. **Binding**: Assigns pod to highest-scoring node

**Example:**
```yaml
# Pod with resource requests
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
  nodeSelector:
    disktype: ssd
```

#### Controller Manager

The **Controller Manager** (`kube-controller-manager`) runs multiple controllers that **watch cluster state** and make changes to achieve desired state.

**Key Controllers:**
- **Deployment Controller**: Manages ReplicaSets and rolling updates
- **ReplicaSet Controller**: Ensures desired number of pod replicas
- **Node Controller**: Monitors node health and availability
- **Service Account Controller**: Creates default service accounts
- **Endpoint Controller**: Manages service endpoints
- **Namespace Controller**: Handles namespace lifecycle

**Control Loop Pattern:**
```
1. Watch current state (via API Server)
2. Compare with desired state
3. Take action if difference exists
4. Repeat continuously
```

**Controller Actions:**
- **Create**: New pods when replicas < desired
- **Delete**: Excess pods when replicas > desired
- **Update**: Rolling updates for deployments
- **Monitor**: Node failures and pod health

#### ETCD

**ETCD** is a distributed key-value store that serves as the **cluster's database** storing all cluster state and configuration.

**Key Characteristics:**
- **Distributed**: Runs on multiple nodes for high availability
- **Consistent**: Uses Raft consensus algorithm (Leader, Follower, Election)
- **Persistent**: All cluster data survives restarts
- **Watchable**: Components can watch for changes

**Data Stored in ETCD:**
- **Cluster Configuration**: Node information, network settings
- **Resource Definitions**: Pods, Services, ConfigMaps, Secrets
- **Resource State**: Current status of all objects
- **Metadata**: Labels, annotations, resource versions

**ETCD Structure:**
```
/registry/
├── pods/default/nginx-pod
├── services/default/kubernetes
├── deployments/default/web-app
├── configmaps/kube-system/kubeadm-config
└── secrets/default/mysecret
```

**High Availability:**
- **Cluster Size**: Typically 3 or 5 nodes (odd numbers)
- **Quorum**: Majority must be available (2/3 or 3/5)
- **Leader Election**: One leader handles writes
- **Automatic Failover**: New leader elected if current fails

**Performance Considerations:**
- **Network Latency**: Keep ETCD nodes close together
- **Disk I/O**: Use SSDs for better performance
- **Backup Strategy**: Regular snapshots essential
- **Monitoring**: Watch for high latency and failed requests

#### Components Interaction

The control plane components work together in a coordinated manner to manage the cluster:

**Flow of Operations:**
```
User

kubectl apply
      |
      v
+----------------+
| API Server     |
+----------------+
      |
      v
Authentication
      |
Authorization
      |
Admission
      |
Validation
      |
      v
+----------------+
| etcd           |
+----------------+
      ^
      |
Controller Manager watches
      |
Deployment Controller
      |
ReplicaSet Controller
      |
Creates Pods
      |
      v
+----------------+
| API Server     |
+----------------+
      |
      v
Scheduler watches
      |
Find node
      |
Assign nodeName
      |
      v
API Server
      |
      v
kubelet watches
      |
Container Runtime
      |
CNI Plugin
      |
Pod starts
      |
      v
API Server
      |
      v
etcd updated
```

**Example Complete Flow:**

**User applies deployment with `replicas: 3`:**

**1️⃣ Deployment Controller creates pods**
```
kubectl apply deployment.yaml (replicas: 3)
    ↓
• API Server validates & stores deployment in ETCD
• Deployment Controller sees new deployment
• Deployment Controller → creates ReplicaSet via API Server
• ReplicaSet Controller creates 3 pods via API Server
• Pods appear in API Server with no assigned node (nodeName: null)
```

**2️⃣ Scheduler sees unscheduled pods**
```
• Scheduler watches API Server continuously
• Finds pods with no nodeName field
• Scheduler evaluates nodes → selects optimal node → binds pod to node
• Pod now has: spec.nodeName: worker-1
• API Server updates pod assignment in ETCD
```

**3️⃣ Kubelet starts the pod**
```
• Kubelet on worker-1 watches API Server for pods assigned to its node
• Kubelet reads pod spec → pulls images → creates containers
• Pod status progression: Pending → Running
• API Server reflects the running state in ETCD
```

**4️⃣ Controller Manager reacts to pod/node status changes**
```
Controllers continuously listen for changes:

• Node Controller: If node dies → marks pods as "lost"
• ReplicaSet Controller: If pod dies → creates replacement pod
• Deployment Controller: Updates ReplicaSets during rolling updates

This feeds back into the loop → Scheduler schedules new pods again
```

----

### Worker Node Components

Worker nodes run the actual application workloads and contain three essential components that work together to manage pods and containers.

#### Kubelet

The **Kubelet** is the primary **node agent** that communicates with the API Server and manages containers on the worker node.

**Key Functions:**
- **Pod Lifecycle Management**: Creates, monitors, and destroys pods
- **API Server Communication**: Registers node and reports pod status
- **Container Management**: Works with container runtime to manage containers
- **Resource Monitoring**: Tracks CPU, memory, and storage usage
- **Health Checks**: Performs liveness and readiness probes

**Kubelet Responsibilities:**
```
┌─ API Server ─┐    ┌─ Worker Node ─┐
│              │◄──►│   Kubelet     │
│ Pod Specs    │    │      │        │
│ Node Status  │    │      ▼        │
└──────────────┘    │ Container     │
                    │ Runtime       │
                    │      │        │
                    │      ▼        │
                    │ Running Pods  │
                    └───────────────┘
```

**Kubelet Managed Components:**
- **User Workloads**: Application pods scheduled by the control plane
- **Node-level Agents**: Metrics exporters, log collectors, monitoring agents

#### Kube-proxy

The **Kube-proxy** is responsible for **network proxying** and implementing Kubernetes Service networking on each node.

**Key Functions:**
- **Service Discovery**: Maintains network rules for Services
- **Load Balancing**: Distributes traffic across multiple pod endpoints
- **Network Rule Management**: Updates iptables/IPVS rules. Updates rules when pods are added/removed
- **Cluster Networking**: Enables pod-to-service communication

**Service Traffic Flow:**
```
Pod A → ClusterIP:80 → kube-proxy → iptables rules → Pod B (backend)
                                   ├─────────────────► Pod C (backend)
                                   └─────────────────► Pod D (backend)
```

**Kube-proxy Modes:**
- **iptables mode**: Uses iptables rules (default, good performance)
- **IPVS mode**: Uses Linux IP Virtual Server (better for large clusters)

kube-proxy itself chooses the mode at startup, but the cluster admin can control this via:

- **`--proxy-mode=...` flag**: Direct command line specification
- **kube-proxy ConfigMap**: Cluster-wide configuration
- **System-level support**: IPVS modules, ipvsadm availability

If no mode is explicitly forced:
1. kube-proxy picks **IPVS** if available
2. Else **iptables**  
3. Else **userspace** (deprecated)

---

#### Container Runtime

The **Container Runtime** is the software responsible for **running containers** on the worker node.

**Key Functions:**
- **Container Lifecycle**: Create, start, stop, and delete containers
- **Image Management**: Pull, store, and manage container images
- **Resource Isolation**: Implement cgroups and namespaces
- **Container Networking**: Set up network interfaces for containers

**Container Runtime Interface (CRI):**
- **Standard Interface**: Kubelet uses CRI to communicate with container runtimes
- **Runtime Agnostic**: Kubernetes works with any CRI-compliant runtime
- **Pluggable Architecture**: Easy to switch between different runtimes

**Popular Container Runtimes:**
1. containerd - Most widely used (Docker Desktop, cloud providers), Lightweight, daemon-based, Strong OCI compliance
2. CRI-O - Specifically designed for Kubernetes
3. Docker Engine (via dockershim - deprecated) - Historical default (removed in K8s 1.24+)

## Networking

Kubernetes networking enables communication between pods, services, and external traffic. It follows a flat network model where every pod gets its own IP address.

### CNI (Container Network Interface)

**CNI** is a specification and library for configuring network interfaces in Linux containers.

**Key Concepts:**
- **Plugin-based**: Different network implementations (Calico, Flannel, Cilium)
- **Pod Networking**: Assigns IP addresses to pods
- **Network Policies**: Controls traffic between pods
- **Multi-host**: Enables pod communication across nodes

**CNI Workflow:**
```
Kubelet → CNI Plugin → Network Setup → Pod gets IP → Pod ready
```

### Pod-to-Pod Communication

Kubernetes networking model ensures **every pod can communicate with every other pod** without NAT.

#### Same Node Communication

**Direct Communication** via the node's internal network:
```
Pod A (10.244.1.2) → Node Bridge → Pod B (10.244.1.3)
```

**Process:**
1. Pods share the same node network namespace
2. Communication via Linux bridge (docker0/cni0)
3. Direct Layer 2 switching
4. No additional routing needed

#### Cross-Node Communication

**Routed Communication** across different nodes:
```
Pod A (Node1: 10.244.1.2) → Node Network → Pod B (Node2: 10.244.2.3)
```

**Process:**
1. CNI plugin configures routing between nodes
2. Overlay networks (VXLAN) or direct routing
3. Each node has unique pod CIDR range
4. Inter-node routing via CNI implementation

**Network Flow:**
```
Pod A → Node1 Bridge → Node1 Interface → Network → Node2 Interface → Node2 Bridge → Pod B
```

### Services
Check in detail in [Kubernetes services in detail](./k8s_services.md)

#### Ingress

**Ingress** provides **HTTP/HTTPS routing** to services from outside the cluster.

**Key Features:**
- **Layer 7 Load Balancing**: Host and path-based routing
- **SSL Termination**: HTTPS certificate management
- **Single Entry Point**: One load balancer for multiple services
- **Cost Effective**: Shared infrastructure

**Ingress Components:**
- **Ingress Controller**: Implements the ingress (NGINX, Traefik, HAProxy)
- **Ingress Resource**: Configuration defining routing rules

**Example Traffic Flow:**
```
Internet → Load Balancer → Ingress Controller → Service → Pods
         (myapp.com/api → backend-service)
         (myapp.com/ → frontend-service)
```

### CNI Plugins

Popular CNI implementations with different features and performance characteristics:

#### Calico
- **Network Policies**: Advanced security rules
- **BGP Routing**: Scalable pure L3 approach
- **Performance**: High performance, no overlay
- **Use Case**: Security-focused, large clusters

#### Flannel
- **Simplicity**: Easy setup and configuration
- **Overlay Network**: VXLAN-based
- **Performance**: Good for small/medium clusters
- **Use Case**: Simple deployments, getting started

#### Cilium
- **eBPF-based**: Kernel-level networking and security
- **Observability**: Advanced traffic monitoring
- **Performance**: High performance with eBPF
- **Use Case**: Modern clusters, security, observability

**CNI Selection Criteria:**
```
┌─────────────┬──────────────┬──────────────┬──────────────┐
│   Feature   │   Calico     │   Flannel    │   Cilium     │
├─────────────┼──────────────┼──────────────┼──────────────┤
│ Complexity  │ Medium       │ Low          │ High         │
│ Performance │ High         │ Medium       │ Very High    │
│ Security    │ Advanced     │ Basic        │ Advanced     │
│ Observability│ Good        │ Basic        │ Excellent    │
└─────────────┴──────────────┴──────────────┴──────────────┘
```

### DNS Resolution

**Step-by-step DNS resolution inside Kubernetes:**

**1. App issues a DNS query**
- The application in the pod makes a DNS request (e.g., `my-service`) using the pod's resolver

**2. Pod uses /etc/resolv.conf**
- The pod's `/etc/resolv.conf` points to the CoreDNS ClusterIP (e.g., `nameserver 10.96.0.10`) and search domains (so short names are expanded)

**3. Query sent to CoreDNS ClusterIP**
- The packet goes to the ClusterIP for kube-dns/CoreDNS. kube-proxy routes the packet to one of the CoreDNS pods

**4. CoreDNS receives the query**
- CoreDNS uses its kubernetes plugin and reads Service/Endpoint (or EndpointSlice) info from the Kubernetes API to resolve names

**5. CoreDNS returns the answer**
- **For a normal Service** → returns the Service ClusterIP (A record)
- **For a headless Service** → returns the backend pod IP(s)
- **For unknown names** → CoreDNS forwards to external DNS (if configured)

**6. Pod connects to returned IP**
- The application uses the IP returned by CoreDNS to open the connection

---

## ETCD & Cluster State

### What is ETCD

### Data Stored in ETCD

### ETCD Backup

### ETCD Restore

---

## Pods, Deployments, ReplicaSets

### Desired State vs Actual State

### RollingUpdate Strategy

### Container Crash Handling

---

## Scheduling

Kubernetes provides multiple mechanisms to control pod placement and scheduling decisions. The following table compares the main scheduling strategies:

| Feature | nodeSelector | Node Affinity | Taints & Tolerations |
|---------|-------------|---------------|----------------------|
| **Purpose** | Select a node with a specific label | Advanced node selection | Prevent unwanted Pods from running on a node |
| **Configured On** | Pod | Pod | Taint on Node, Toleration on Pod |
| **Decision Made By** | Pod says "I want this node" | Pod says "I require/prefer this node" | Node says "Don't schedule Pods here unless they tolerate me" |
| **Rule Type** | Hard only | Hard or Soft | Restriction with exceptions |
| **Scheduling Behavior** | Pod is scheduled only on matching nodes | Pod is scheduled based on required or preferred rules | Pod is blocked unless it has a matching toleration |
| **Supports Expressions** | No | Yes (In, NotIn, Exists, Gt, Lt) | No (matches taints) |
| **Typical Use Case** | GPU nodes, SSD nodes | Zone, instance type, GPU preference | Reserve GPU, database, or master nodes |

### Node Selection

#### nodeSelector

#### nodeAffinity

### Taints and Tolerations

### Pod Priority and Preemption

**Pod Priority and Preemption** allow you to specify the relative importance of pods. Higher-priority pods can preempt (remove) lower-priority pods to acquire resources.

**Key Points:**
- **PriorityClass**: Defines priority levels (e.g., critical, standard, low)
- **Pod Assignment**: Pods reference a PriorityClass in their spec
- **Preemption**: High-priority pending pods can evict lower-priority running pods
- **Graceful Termination**: Evicted pods get a grace period before termination
- **Use Cases**: Ensure critical workloads always have resources

**Example:**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
```

### Workload Types

#### DaemonSet vs Deployment

---
