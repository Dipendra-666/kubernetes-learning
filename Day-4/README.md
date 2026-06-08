# Why Do We Need Ingress If We Already Have a LoadBalancer Service?

A Kubernetes Service of type `LoadBalancer` can expose an application to the internet, but it has some limitations in a microservices environment.

## 1. Cost Optimization

In a microservices architecture, if we have multiple services and expose each one using a `LoadBalancer` Service, the cloud provider will typically create a separate external load balancer and public IP address for every service.

For example:

```text
Frontend Service  -> LoadBalancer -> Public IP 1
User Service      -> LoadBalancer -> Public IP 2
Order Service     -> LoadBalancer -> Public IP 3
Payment Service   -> LoadBalancer -> Public IP 4
```

As the number of services grows, infrastructure costs increase and management becomes more complex.

With Ingress, all incoming traffic enters through a single external load balancer and is then routed to the appropriate services inside the cluster.

```text
Internet
    ↓
Load Balancer
    ↓
Ingress Controller
    ↓
┌───────────────┬───────────────┬───────────────┐
│ User Service  │ Order Service │ Payment Service │
└───────────────┴───────────────┴───────────────┘
```

This significantly reduces the number of public IPs and external load balancers required.

---

## 2. Advanced Layer 7 (HTTP/HTTPS) Traffic Management

A `LoadBalancer` Service primarily operates at **Layer 4 (Transport Layer)** and forwards traffic based on:

- IP Address
- Port
- Protocol (TCP/UDP)

Ingress operates at **Layer 7 (Application Layer)** and understands HTTP/HTTPS traffic, enabling advanced routing and traffic management features such as:

- Path-based routing (`/api`, `/orders`, `/users`)
- Host-based routing (`api.company.com`, `admin.company.com`)
- SSL/TLS termination
- Sticky sessions
- Canary deployments
- Weighted traffic routing
- Rate limiting
- Authentication and authorization

### Example: Path-Based Routing

```text
company.com/api     -> API Service
company.com/orders  -> Order Service
company.com/users   -> User Service
```

Ingress can intelligently inspect incoming HTTP requests and route them to the correct backend service based on the URL path or hostname.

---

## Conclusion

While a `LoadBalancer` Service is sufficient for exposing a single application, it becomes expensive and difficult to manage in a microservices architecture.

Ingress provides:

- A single entry point for the cluster
- Reduced infrastructure cost
- Centralized traffic management
- Advanced Layer 7 routing capabilities

This makes Ingress the preferred solution for exposing and managing multiple services in production Kubernetes environments.

---

## Interview Answer

> We use Ingress because a LoadBalancer Service mainly provides Layer 4 load balancing and exposes a single service externally. In a microservices architecture, creating a LoadBalancer for every service increases cloud costs and management overhead. Ingress provides a single entry point to the cluster and supports advanced Layer 7 features such as path-based routing, host-based routing, SSL termination, sticky sessions, canary deployments, and rate limiting. This makes it more scalable, cost-effective, and suitable for production environments.







# Types of Ingress in Kubernetes

Ingress allows us to route external HTTP/HTTPS traffic to services inside a Kubernetes cluster based on defined rules.

## 1. Path-Based Routing

Path-based routing forwards traffic to different services based on the URL path.

### Example

```text
company.com/api      -> API Service
company.com/orders   -> Order Service
company.com/users    -> User Service
```

### Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  rules:
  - host: company.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80

      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
```

### Use Case

Commonly used in microservices architectures where multiple services share the same domain.

---

## 2. Host-Based Routing

Host-based routing forwards traffic based on the hostname or domain name.

### Example

```text
api.company.com      -> API Service
admin.company.com    -> Admin Service
shop.company.com     -> Shop Service
```

### Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
spec:
  rules:
  - host: api.company.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80

  - host: admin.company.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

### Use Case

Useful when different applications need their own subdomains.

---

## 3. Wildcard Host Routing

Wildcard routing allows one rule to match multiple subdomains.

### Example

```text
dev.company.com
test.company.com
staging.company.com
```

Instead of creating separate rules for each subdomain:

```yaml
host: "*.company.com"
```

### Traffic Flow

```text
dev.company.com      -> Application
test.company.com     -> Application
staging.company.com  -> Application
```

### Use Case

Frequently used for:

- Multi-tenant applications
- Development environments
- SaaS platforms

---

## 4. TLS / HTTPS Ingress

Ingress can terminate SSL/TLS connections and serve applications over HTTPS.

### Without TLS

```text
User
  ↓
HTTP
  ↓
Ingress
  ↓
Application
```

### With TLS

```text
User
  ↓
HTTPS
  ↓
Ingress
  ↓
Application
```

### Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
spec:
  tls:
  - hosts:
    - company.com
    secretName: company-tls-secret

  rules:
  - host: company.com
```

### Use Case

Used to:

- Encrypt traffic
- Secure user credentials
- Protect sensitive data

---

## 5. Default Backend Routing

A default backend handles requests that do not match any defined Ingress rule.

### Example

```text
company.com/api      -> API Service
company.com/orders   -> Order Service

company.com/random-page
        ↓
Default Backend
```

### Use Case

Useful for:

- Custom 404 pages
- Error handling
- Redirecting unknown requests

---

# Interview Summary

Ingress routing can be classified into:

1. **Path-Based Routing** – Routes traffic based on URL paths.
2. **Host-Based Routing** – Routes traffic based on domain names.
3. **Wildcard Routing** – Routes multiple subdomains using a wildcard pattern.
4. **TLS/HTTPS Ingress** – Provides SSL/TLS termination and secure communication.
5. **Default Backend Routing** – Handles unmatched requests.

These routing capabilities make Ingress a powerful Layer 7 traffic management solution for Kubernetes applications.