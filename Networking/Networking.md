# 🌐 DevOps Networking – Complete Guide (Single Server → Kubernetes on EKS)

> **Purpose of this README**
> This document is designed as **long‑term reference notes + interview preparation material**.
> All concepts are explained using **real‑life analogies, simple examples, ASCII architecture diagrams**, and finally **mapped step‑by‑step to AWS EKS**.

---

## 🧠 BIG PICTURE – HOW EVERYTHING CONNECTS

```
User
 │
 │  (DNS Resolution)
 ▼
Internet
 │
 │  (Public IP)
 ▼
Load Balancer / Ingress
 │
 │  (Routing Rules)
 ▼
Services (Kubernetes)
 │
 │  (East–West Traffic)
 ▼
Pods (Microservices)
 │
 ▼
Database (Private Network)
```

Keep this flow in mind — every networking concept exists **to control or protect one hop in this journey**.

---

# PHASE 1️⃣ – SINGLE SERVER ON THE INTERNET

## 1. Single Server

**Real Life Example:**
A single shop on a highway.

* One server
* One application
* One public IP

```
User ───► Server (App)
```

---

## 2. How People Find a Server on the Internet

### Problem

Humans cannot remember IP addresses.

### Solution

**DNS (Domain Name System)**

**Real Life Analogy**

| Internet    | Real Life    |
| ----------- | ------------ |
| IP Address  | GPS Location |
| Domain Name | Shop Name    |
| DNS         | Google Maps  |

---

## 3. IP Address (Public vs Private)

### Public IP

* Globally unique
* Accessible from internet
* Example:

```
13.234.56.78
```

### Private IP

* Used internally
* Not reachable from internet
* Common ranges:

```
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

**Key Rule:**

> Internet traffic uses **Public IP**, internal traffic uses **Private IP**

---

## 4. What is DNS (Detailed)

### DNS Flow

```
User enters google.com
        │
        ▼
Local DNS Resolver
        │
        ▼
Authoritative DNS Server
        │
        ▼
Public IP Address
```

### Why DNS is Required

* Human‑friendly access
* IP can change without affecting users
* Enables load balancing & failover

---

# PHASE 2️⃣ – MULTIPLE APPLICATIONS ON A SINGLE SERVER

## 5. What are Ports?

**Real Life Analogy**

| Server | Apartment Building |
| ------ | ------------------ |
| Port   | Flat Number        |
| App    | Resident           |

```
Server IP: 13.234.56.78

13.234.56.78:80    → Frontend
13.234.56.78:8080  → Backend
13.234.56.78:3306  → Database
```

### Standard Ports (Important for Interviews)

| Port | Service    |
| ---- | ---------- |
| 80   | HTTP       |
| 443  | HTTPS      |
| 22   | SSH        |
| 3306 | MySQL      |
| 5432 | PostgreSQL |

---

# PHASE 3️⃣ – SECURITY & SEGMENTATION

## 6. Why Security and Segmentation Are Needed

**Real Life:**
A bank has public hall, staff area, and vault.

Without segmentation:

* One breach = full system compromised

---

## 7. Subnets (with IP Ranges)

```
VPC: 10.0.0.0/16

Public Subnet:  10.0.1.0/24
Private Subnet: 10.0.2.0/24
```

### Why Subnets?

* Isolation
* Security control
* Traffic management

---

## 8. CIDR Notation (Very Important)

### What is CIDR?

CIDR defines **how many IPs** a network has.

```
10.0.1.0/24
```

* `/24` → 256 IPs
* `/16` → 65,536 IPs

**Real Life:**
CIDR = number of houses in a society

---

## 9. Routing

Routing decides **where traffic should go next**.

### Route Table Example

```
Destination     Target
0.0.0.0/0       Internet Gateway
10.0.0.0/16     Local
```

**Router = Traffic Police**

---

## 10. Firewalls

### Host Firewall

* Runs on server
* Example: iptables, ufw

### Network Firewall

* Centralized control
* Protects entire subnet

---

## 11. Layered Security (Defense in Depth)

```
Internet
 │
Network Firewall
 │
Subnet Rules
 │
Host Firewall
 │
Application Auth
```

---

## 12. Secured Zones

| Zone       | Purpose        |
| ---------- | -------------- |
| Public     | Load balancers |
| Private    | App servers    |
| Restricted | Databases      |

---

# PHASE 4️⃣ – PRIVATE SERVERS NEED INTERNET

## 13. NAT (Network Address Translation)

### Problem

Private servers need internet but should not be accessible.

### Solution

**NAT Gateway**

```
Private Server ─► NAT ─► Internet
Internet ─X─► Private Server
```

**Real Life:**
Office receptionist calling outside on behalf of employees

---

# PHASE 5️⃣ – MOVING TO CLOUD (AWS VPC)

## 14. What is VPC?

**VPC = Your private data center inside AWS**

### Why VPC?

* Isolation
* Security
* Full network control

---

## 15. VPC Architecture (ASCII)

```
AWS VPC (10.0.0.0/16)
 ├─ Public Subnet
 │    └─ Load Balancer
 │
 ├─ Private Subnet
 │    └─ App Servers
 │
 └─ Restricted Subnet
      └─ Database
```

---

## 16. Gateways & Route Tables

* Internet Gateway → Public access
* NAT Gateway → Private outbound access

---

## 17. Security Groups

* Instance‑level firewall
* Stateful

```
Allow 80 from 0.0.0.0/0
Allow 22 from My IP
```

---

# PHASE 6️⃣ – CONTAINER NETWORKING

## 18. Microservices Architecture

```
Auth Service
Payment Service
Order Service
```

Each service runs independently.

---

## 19. Docker Bridge Network

```
Host
 ├─ Container A (172.17.0.2)
 ├─ Container B (172.17.0.3)
```

---

## 20. Port Binding (Similar to NAT)

```
Host:8080 ─► Container:80
```

---

## 21. Overlay Networks

* Virtual network across multiple hosts
* Used in Docker Swarm & Kubernetes

---

# PHASE 7️⃣ – KUBERNETES NETWORKING

## 22. Kubernetes Networking Model

* Every Pod gets IP
* Pods communicate directly
* Services provide stable access

---

## 23. Load Balancer vs Ingress

| Load Balancer      | Ingress                  |
| ------------------ | ------------------------ |
| Layer 4/7          | Layer 7 only             |
| One LB per service | One LB for many services |
| Expensive          | Cost‑effective           |

---

## 24. Ingress Traffic Flow

```
User
 ↓
DNS
 ↓
Cloud Load Balancer
 ↓
Ingress Controller
 ↓
Service
 ↓
Pod
```

---

## 25. North‑South vs East‑West Traffic

| Traffic Type | Meaning           |
| ------------ | ----------------- |
| North‑South  | User → App        |
| East‑West    | Service → Service |

---

## 26. Service Mesh

### What is Service Mesh?

Dedicated layer to manage **service‑to‑service communication**.

**Examples:** Istio, Linkerd

Provides:

* mTLS
* Traffic control
* Observability

---

## 27. Zero Trust Networking

**Rule:**

> Never trust, always verify

* No implicit trust
* Identity‑based access
* Strong authentication everywhere

---

# PHASE 8️⃣ – EKS MAPPING (STEP‑BY‑STEP)

## Step 1: Create VPC

* CIDR: 10.0.0.0/16

## Step 2: Create Subnets

* Public Subnet → Load Balancer
* Private Subnet → EKS Nodes

## Step 3: Attach Internet Gateway

## Step 4: Create NAT Gateway

## Step 5: Configure Route Tables

## Step 6: Create EKS Cluster

## Step 7: Deploy Ingress Controller

## Step 8: Deploy Services & Pods

---

# 🎯 INTERVIEW PREPARATION – QUICK REVISION

| Topic          | Simple Definition            |
| -------------- | ---------------------------- |
| IP Address     | Unique machine identifier    |
| DNS            | Name to IP resolver          |
| CIDR           | IP range definition          |
| Port           | Application entry point      |
| Subnet         | Network partition            |
| Routing        | Traffic direction rules      |
| Firewall       | Traffic filter               |
| NAT            | Private to public access     |
| VPC            | Private cloud network        |
| Security Group | Instance firewall            |
| Load Balancer  | Traffic distributor          |
| Ingress        | HTTP routing layer           |
| North‑South    | External traffic             |
| East‑West      | Internal traffic             |
| Service Mesh   | Secure service communication |
| Zero Trust     | Verify everything            |

---

✅ **This README can be directly used for GitHub reference and interview preparation.**
