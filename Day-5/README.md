# Kubernetes Deployment Strategies

## Why Do We Need Deployment Strategies?

In a production environment, applications are continuously updated to release new features, fix bugs, and patch security vulnerabilities.

Consider an application running with 5 replicas:

```text
frontend-v1

Pod-1
Pod-2
Pod-3
Pod-4
Pod-5
```

Now a new version (`v2`) needs to be deployed.

If Kubernetes deletes all existing Pods and creates new Pods at once:

```text
Delete v1 Pods
       ↓
No Running Application
       ↓
Create v2 Pods
```

users will experience downtime.

Deployment strategies solve this problem by providing controlled mechanisms to upgrade applications while maintaining availability and minimizing risk.

### Goals of Deployment Strategies

- Minimize or eliminate downtime
- Maintain application availability
- Allow safe rollback
- Reduce deployment risk
- Ensure smooth user experience

---

# Rolling Update Strategy

## What is Rolling Update?

Rolling Update is the default deployment strategy in Kubernetes.

Instead of replacing all Pods simultaneously, Kubernetes gradually replaces old Pods with new Pods.

During the deployment, both versions may coexist temporarily.

```text
Before Update

Pod-1 (v1)
Pod-2 (v1)
Pod-3 (v1)
Pod-4 (v1)
```

```text
During Update

Pod-1 (v2)
Pod-2 (v2)
Pod-3 (v1)
Pod-4 (v1)
```

```text
After Update

Pod-1 (v2)
Pod-2 (v2)
Pod-3 (v2)
Pod-4 (v2)
```

This ensures that users can continue accessing the application during the update.

---

## How Rolling Update Works Internally

A Deployment never directly manages Pods.

The hierarchy is:

```text
Deployment
      ↓
ReplicaSet
      ↓
Pods
```

Initially:

```text
Deployment
      ↓
ReplicaSet-v1
      ↓
4 Pods
```

When a new image version is deployed:

```yaml
image: nginx:v2
```

Kubernetes creates a new ReplicaSet.

```text
Deployment
      ├── ReplicaSet-v1
      └── ReplicaSet-v2
```

The Deployment Controller then:

1. Scales up ReplicaSet-v2
2. Scales down ReplicaSet-v1

until all Pods are running the new version.

---

## maxSurge

### Definition

`maxSurge` controls how many extra Pods Kubernetes can create above the desired replica count during an update.

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
```

Current State:

```text
Desired Replicas = 4
```

Normally:

```text
Pod-1
Pod-2
Pod-3
Pod-4
```

With:

```yaml
maxSurge: 1
```

Kubernetes can temporarily create:

```text
Pod-1
Pod-2
Pod-3
Pod-4
Pod-5
```

Total:

```text
5 Pods
```

during the deployment.

### Why Do We Need maxSurge?

Without maxSurge:

```text
Delete Old Pod
      ↓
Create New Pod
```

Application capacity temporarily decreases.

With maxSurge:

```text
Create New Pod
      ↓
Delete Old Pod
```

Application capacity remains stable.

---

## maxUnavailable

### Definition

`maxUnavailable` defines the maximum number of Pods that can be unavailable during the update.

Example:

```yaml
strategy:
  rollingUpdate:
    maxUnavailable: 1
```

Current Replicas:

```text
4
```

Kubernetes guarantees:

```text
At least 3 Pods remain available
```

throughout the deployment.

### Why Do We Need maxUnavailable?

It prevents too many Pods from being taken down simultaneously.

This helps maintain application availability during updates.

---

## Example

```yaml
replicas: 10

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 2
    maxUnavailable: 1
```

Meaning:

```text
Desired Pods = 10

Maximum Pods Running = 12

Minimum Available Pods = 9
```

during the deployment process.

---

## Advantages

- Zero or near-zero downtime
- Native Kubernetes support
- Efficient resource utilization
- Automatic rollback support

---

## Disadvantages

- Old and new versions run simultaneously
- Version compatibility issues may occur
- Bugs may affect users before detection

---

# Blue-Green Deployment

## What is Blue-Green Deployment?

Blue-Green deployment uses two identical environments.

### Blue Environment

Current production environment.

```text
Blue (v1)

Frontend
Backend
Database
```

### Green Environment

New version environment.

```text
Green (v2)

Frontend
Backend
Database
```

Only one environment receives production traffic.

---

## Deployment Process

### Step 1

Users access Blue.

```text
Users
   ↓
Blue (v1)
```

---

### Step 2

Deploy new version to Green.

```text
Blue (v1)
Green (v2)
```

---

### Step 3

Perform:

- Smoke Testing
- Health Checks
- Functional Testing

on Green.

---

### Step 4

Switch traffic.

```text
Users
   ↓
Green (v2)
```

Blue remains available as a backup.

---

## Rollback

If issues are detected:

```text
Users
   ↓
Blue (v1)
```

Traffic can be switched back almost instantly.

This is the biggest advantage of Blue-Green deployment.

---

## Advantages

- Near-zero downtime
- Instant rollback
- Safe production releases
- Easy testing before cutover

---

## Disadvantages

- Requires duplicate infrastructure
- Higher cloud costs
- Environment synchronization challenges

---

## Use Cases

- Banking Applications
- E-Commerce Platforms
- Enterprise Production Systems
- Mission-Critical Workloads

---

# Canary Deployment

## What is Canary Deployment?

Canary deployment gradually exposes the new version to a small percentage of users before rolling it out to everyone.

Instead of sending all traffic to the new version:

```text
100% → v1
```

traffic is gradually shifted.

---

## Deployment Process

### Stage 1

```text
95% → v1
5%  → v2
```

Monitor:

- Error Rate
- Latency
- CPU Usage
- Memory Usage
- Business Metrics

---

### Stage 2

If healthy:

```text
80% → v1
20% → v2
```

---

### Stage 3

```text
50% → v1
50% → v2
```

---

### Stage 4

```text
100% → v2
```

---

## Why Canary?

Suppose:

```text
10 Million Users
```

and a bug exists in v2.

With Blue-Green:

```text
10 Million Users may be impacted.
```

With Canary:

```text
Only a small percentage of users are impacted.
```

This significantly reduces deployment risk.

---

## How Is Canary Implemented?

Kubernetes Deployments do not natively support percentage-based traffic routing.

Canary deployments are commonly implemented using:

- Argo Rollouts
- Istio
- Linkerd
- NGINX Ingress
- AWS Load Balancer Weighted Routing

---

## Advantages

- Lowest deployment risk
- Real-world validation
- Gradual rollout
- Better monitoring opportunities

---

## Disadvantages

- More complex implementation
- Requires traffic management tools
- Requires advanced observability

---

# Deployment Strategy Comparison

| Feature | Rolling Update | Blue-Green | Canary |
|----------|----------|----------|----------|
| Downtime | None | None | None |
| Rollback Speed | Medium | Instant | Fast |
| Infrastructure Cost | Low | High | Medium |
| Complexity | Low | Medium | High |
| Risk Level | Medium | Low | Lowest |
| Resource Usage | Low | High | Medium |

---

# Interview Summary

Rolling Update is Kubernetes' default deployment strategy that gradually replaces old Pods with new Pods using ReplicaSets. Parameters such as `maxSurge` and `maxUnavailable` control deployment speed and availability.

Blue-Green deployment maintains two separate production environments and switches traffic between them, providing instant rollback capabilities.

Canary deployment gradually exposes a new version to a subset of users before a full rollout, minimizing deployment risk and enabling real-world validation.