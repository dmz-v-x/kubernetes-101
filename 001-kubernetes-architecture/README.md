# Kubernetes Architecture — Complete Updated Guide (Step-by-Step, Beginner Friendly)

This blog explains **Kubernetes architecture** with the **latest information from official sources** and **industry-standard documentation**. Everything is explained in **simple, ordered steps** so even beginners can understand.

---

## What Is Kubernetes?

- **Kubernetes (K8s)** is an open-source system for automating deployment, scaling, and management of containerized applications. 
- It originated from Google’s internal system **Borg** and is now maintained by the **Cloud Native Computing Foundation (CNCF)**. 

Kubernetes helps you run containerized applications at scale with:
- **Self-healing**
- **Autoscaling**
- **Declarative configuration**
- **High availability**

---

## High-Level Architecture Overview

A Kubernetes cluster consists of:

1. **Control Plane (Brain of the Cluster)**
2. **Worker Nodes (Data Plane / Worker Machines)**

Every component has a specific role to make the cluster reliable and scalable.

---

## 1. Node — Basic Unit in Kubernetes

A **Node** is simply a machine (virtual or physical) where workloads run. 

There are **two types of nodes**:

### Control Plane Node (also called Master)
- Runs the **control plane components**.
- Responsible for **management, scheduling, and orchestration**.
- In production, multiple control plane nodes are used for **high availability**. 

### Worker Nodes
- These nodes run the actual applications inside **Pods**.
- Pods encapsulate containers that run the workloads.
- Workers do not host control plane components.

---

## 2. Kubernetes Control Plane — Cluster Brain

The control plane **manages the entire cluster** by monitoring desired and actual state, making decisions, and enforcing the state across nodes.

### 2.1 API Server (`kube-apiserver`)

The **API Server** is the **central access point** for all Kubernetes communication.

- All requests (from `kubectl`, UI, controllers, or other tools) go through the API server. 
- It performs:
  - **Authentication**
  - **Authorization**
  - **Validation**
  - **Admission control**  
- It exposes the Kubernetes API over HTTP/REST. 

The API server is the only component that reads from and writes to **etcd**. 

---

### 2.2 etcd — Distributed Key-Value Store

**etcd** is the **persistent and consistent key-value datastore** for the cluster. 

- Stores cluster state information like:
  - Pod specs
  - Configs
  - Secrets
  - Service definitions
- Acts as the **single source of truth** for Kubernetes.
- Only the API server communicates directly with etcd. 

For high availability, etcd is run on multiple nodes with **leader election (RAFT)**.

---

### 2.3 Scheduler (`kube-scheduler`)

The Scheduler decides **which node** a Pod should run on.

It looks at:
- Resource requirements (CPU, RAM)
- Node capacity
- Affinity/anti-affinity
- Taints and tolerations
- Policy constraints

Once a node is selected, it updates the Pod object with the assigned node. 

---

### 2.4 Controller Manager (`kube-controller-manager`)

This component runs **multiple controllers** that help maintain the cluster’s desired state. 

Controllers include:
- **Node Controller** — Tracks node health.
- **Replication Controller** — Ensures desired number of Pod replicas.
- **Deployment Controller** — Maintains deployment state.
- **Job Controller** — Handles batch jobs.

Controllers watch etcd via the API server and act to correct any drift from the desired state. 

---

### 2.5 Cloud Controller Manager (`cloud-controller-manager`) (Optional)

Only present in clusters that run on a cloud provider.

It interacts with cloud APIs to:
- Create load balancers
- Manage routes
- Manage storage volumes

This component **decouples cloud-specific logic** from Kubernetes core.

---

## 3. Worker Nodes — Where Applications Run

Worker nodes host everything that actually runs your workloads. 

Each worker node runs:

### 3.1 Kubelet

`kubelet` is an agent that ensures containers in Pod specs are running as expected.

It:
- Receives Pod specs from the API server
- Starts/stops containers via container runtime
- Reports status back to the control plane

---

### 3.2 Kube-proxy

`kube-proxy` enables **networking and service routing** on the node. 

It configures:
- IP tables or eBPF (modern clusters) for routing
- Ensures that services can reach Pods

---

### 3.3 Container Runtime

This is the low-level software that actually **runs containers** on the node.

Examples include:
- containerd
- CRI-O
- Docker (deprecated in modern setups)

Container runtime implements the Kubernetes **Container Runtime Interface (CRI)**.

---

## How Components Work Together

1. **User** (or automation) calls Kubernetes via `kubectl` or API clients.  
2. **API server** receives requests and validates them.  
3. **etcd** stores the desired state.  
4. **Scheduler** assigns workloads to nodes.  
5. **Controller manager** ensures things stay in the desired state.  
6. **Worker nodes** actually run Pods and containers via kubelet and container runtime. 

---

## High Availability

For production:
- Control plane components are often **replicated**.
- etcd runs in an **odd number** of instances for quorum.
- API servers are **load-balanced**.

This ensures no single failure brings down the cluster. 

---

## Summary (Simple)

| Layer | Key Components |
|-------|----------------|
| **Control Plane** | API Server, etcd, Scheduler, Controller Manager, Cloud Controller Manager |
| **Worker Node** | kubelet, kube-proxy, container runtime |

---

## One-Line Mental Model

> Kubernetes architecture separates **control (management)** from **data plane (workloads)** — control plane makes decisions, worker nodes **execute** them. 

