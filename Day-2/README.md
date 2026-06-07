# Kubernetes Interview Question: What is Kops and How is it Different from kubectl and EKS?

## What is Kops?

Kops (Kubernetes Operations) is an open-source tool used to create, manage, upgrade, and maintain Kubernetes clusters, primarily on AWS.

Kops automates both:

1. Infrastructure provisioning
2. Kubernetes cluster installation and lifecycle management

It can automatically create AWS resources such as:

* VPCs
* Subnets
* EC2 Instances
* Security Groups
* Load Balancers
* Auto Scaling Groups

After provisioning the infrastructure, Kops installs and manages Kubernetes components such as:

* API Server
* etcd
* Controller Manager
* Scheduler
* Worker Nodes

In simple terms:

> Kops creates the infrastructure and installs Kubernetes on top of it.

---

## How Kops Works

```text
AWS Resources
      ↓
EC2 Instances
      ↓
Kubernetes Control Plane
(API Server, etcd, Scheduler, Controller Manager)
      ↓
Worker Nodes
      ↓
Applications
```

Kops is responsible for managing the lifecycle of the Kubernetes cluster, including upgrades, scaling, and configuration changes.

---

## Kops vs EKS

The major difference is who manages the Kubernetes control plane.

### Kops (Self-Managed Kubernetes)

```text
AWS Infrastructure
      ↓
Control Plane (Managed by Kops/User)
      ↓
Worker Nodes
```

Responsibilities:

* Manage API Server
* Manage etcd
* Manage Scheduler
* Manage Controller Manager
* Handle upgrades
* Handle control-plane maintenance

Advantages:

* Full control over Kubernetes
* More customization options

Disadvantages:

* Higher operational overhead
* More maintenance responsibility

---

### EKS (Managed Kubernetes)

```text
AWS Infrastructure
      ↓
AWS Managed Control Plane
      ↓
Worker Nodes
```

Responsibilities of AWS:

* API Server
* etcd
* Scheduler
* Controller Manager
* Control-plane high availability
* Control-plane maintenance

Responsibilities of User:

* Applications
* Deployments
* Services
* Worker Nodes
* Monitoring
* Security Policies

Advantages:

* Less operational effort
* Easier upgrades
* Better reliability

Disadvantages:

* Less control over control-plane internals

---

## What is eksctl?

eksctl is a CLI tool used to create and manage EKS clusters.

Example:

```bash
eksctl create cluster --name demo
```

Important:

eksctl does not install Kubernetes itself.

Instead, it communicates with AWS APIs, and AWS provisions the EKS cluster and managed control plane.

Think of it as:

> eksctl is a management tool for EKS.

---

## Kops vs eksctl

| Feature                | Kops        | eksctl + EKS |
| ---------------------- | ----------- | ------------ |
| Creates Infrastructure | Yes         | Yes          |
| Installs Kubernetes    | Yes         | No           |
| Manages Control Plane  | Yes         | AWS          |
| Manages etcd           | Yes         | AWS          |
| Upgrades Control Plane | User/Kops   | AWS          |
| Operational Overhead   | High        | Low          |
| Common Today           | Less Common | Very Common  |

---

## What is kubectl?

kubectl is the command-line tool used to interact with an existing Kubernetes cluster.

Examples:

```bash
kubectl get pods
kubectl get nodes
kubectl apply -f deployment.yaml
kubectl delete pod nginx
```

kubectl does not create Kubernetes clusters.

It only manages resources running inside a cluster.

---

## Kops vs kubectl

### Kops

Used to manage the Kubernetes cluster itself.

Examples:

```bash
kops create cluster
kops update cluster
kops validate cluster
kops upgrade cluster
```

### kubectl

Used to manage applications and resources inside the cluster.

Examples:

```bash
kubectl get pods
kubectl get deployments
kubectl apply -f deployment.yaml
```

---

## Easy Interview Analogy

Imagine Kubernetes is an apartment building.

### Kops = Construction Company

* Buys land
* Builds the building
* Installs electricity and plumbing
* Upgrades the building

### kubectl = Building Manager

* Assigns tenants
* Checks occupied rooms
* Removes tenants
* Manages day-to-day operations

Kops builds and maintains the building.

kubectl manages what happens inside the building.

---

## Do Companies Still Use Kops?

Kops is still used in some legacy or highly customized environments, but it is much less common today.

Most modern organizations use managed Kubernetes services:

* Amazon EKS
* Azure AKS
* Google GKE

This reduces operational overhead and allows teams to focus on applications rather than managing Kubernetes control-plane components.

---

## Interview Answer (Short Version)

"Kops is an open-source Kubernetes cluster management tool that provisions cloud infrastructure and installs Kubernetes on top of it. Unlike EKS, where AWS manages the control plane, Kops manages the entire Kubernetes cluster lifecycle, including the control plane. kubectl is different because it is used to manage workloads and resources inside an already existing Kubernetes cluster. Today, most organizations prefer managed services like EKS, AKS, and GKE, but understanding Kops is still valuable for interviews and legacy environments."












# Kubernetes Interview Question: What is a Pod and How Do You Debug a Failing Pod?

## What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain:

* One container (most common)
* Multiple tightly coupled containers

Containers inside the same Pod:

* Share the same network namespace
* Share the same IP address
* Can communicate using localhost
* Can share storage volumes

Example:

```text
Pod
├── Nginx Container
└── Sidecar Container
```

In Kubernetes, applications do not run directly on nodes. They run inside Pods.

---

## Interview Answer

"A Pod is the smallest deployable unit in Kubernetes. It acts as a wrapper around one or more containers and provides shared networking and storage. Every application running in Kubernetes runs inside a Pod."

---

# How Do You Debug a Failing Pod?

When a Pod is not working, I follow a systematic troubleshooting approach.

---

## Step 1: Check Pod Status

```bash
kubectl get pods
```

Example Output:

```text
NAME         READY   STATUS             RESTARTS
nginx-pod    0/1     CrashLoopBackOff   5
```

Common Status Values:

| Status           | Meaning                   |
| ---------------- | ------------------------- |
| Running          | Pod is healthy            |
| Pending          | Waiting for resources     |
| Error            | Container failed          |
| CrashLoopBackOff | Container keeps crashing  |
| ImagePullBackOff | Unable to pull image      |
| Completed        | Job finished successfully |

---

## Step 2: Describe the Pod

```bash
kubectl describe pod nginx-pod
```

Purpose:

Shows detailed information about the Pod:

* Events
* Scheduling information
* Resource limits
* Container states
* Error messages

Example:

```text
Events:
FailedScheduling
0/3 nodes available
```

This immediately tells us the Pod cannot be scheduled.

---

## Step 3: Check Container Logs

```bash
kubectl logs nginx-pod
```

Purpose:

Displays application logs from inside the container.

Example:

```text
Database connection failed
```

Now we know the application is running but cannot connect to the database.

---

## Step 4: Check Previous Container Logs

For CrashLoopBackOff situations:

```bash
kubectl logs nginx-pod --previous
```

Purpose:

Shows logs from the previous crashed container.

Very useful when containers restart immediately.

---

## Step 5: Access the Container

```bash
kubectl exec -it nginx-pod -- /bin/bash
```

or

```bash
kubectl exec -it nginx-pod -- sh
```

Purpose:

Allows us to enter the container and inspect:

* Configuration files
* Environment variables
* Network connectivity
* Application files

Example:

```bash
env
cat /app/config.yaml
```

---

## Step 6: Verify Deployment Configuration

```bash
kubectl describe deployment my-app
```

Check:

* Image name
* Environment variables
* Replica count
* Resource requests

---

## Step 7: Check Cluster Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Purpose:

Shows recent Kubernetes events.

Useful for:

* Scheduling failures
* Volume mount issues
* Image pull failures

---

# Common Problems and Fixes

## Problem 1: ImagePullBackOff

Check:

```bash
kubectl describe pod nginx-pod
```

Example:

```text
Failed to pull image
```

Possible Causes:

* Wrong image name
* Wrong image tag
* Private registry authentication issue

Fix:

* Verify image name
* Verify image tag
* Configure imagePullSecrets

---

## Problem 2: CrashLoopBackOff

Check:

```bash
kubectl logs nginx-pod
kubectl logs nginx-pod --previous
```

Possible Causes:

* Application crash
* Missing environment variables
* Database connection failure

Fix:

* Correct configuration
* Fix application error
* Verify dependent services

---

## Problem 3: Pod Stuck in Pending

Check:

```bash
kubectl describe pod nginx-pod
```

Possible Causes:

* Insufficient CPU
* Insufficient Memory
* Node Selector mismatch
* Taints and Tolerations issue

Fix:

* Add more resources
* Scale nodes
* Correct scheduling configuration

---

## Problem 4: OOMKilled

Check:

```bash
kubectl describe pod nginx-pod
```

Example:

```text
Reason: OOMKilled
```

Meaning:

Container exceeded memory limit.

Fix:

Increase memory limits:

```yaml
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
```

---

## Problem 5: Liveness Probe Failure

Check:

```bash
kubectl describe pod nginx-pod
```

Example:

```text
Liveness probe failed
```

Fix:

* Correct probe endpoint
* Increase timeout values
* Ensure application starts properly

---

# Real Interview Answer (2-Minute Version)

"If a Pod is failing, I first run `kubectl get pods` to identify its status. Then I use `kubectl describe pod <pod-name>` to inspect events, scheduling information, container state, and Kubernetes-level errors. Next, I use `kubectl logs <pod-name>` to view application logs and identify runtime issues. If the container is restarting, I check `kubectl logs --previous` to see logs from the last crashed instance. If needed, I use `kubectl exec -it <pod-name> -- bash` to enter the container and verify configuration, environment variables, or network connectivity. Based on the findings, I troubleshoot common issues such as ImagePullBackOff, CrashLoopBackOff, Pending state, OOMKilled events, or probe failures."
