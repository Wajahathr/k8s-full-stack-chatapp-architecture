# 🚀 Enterprise-Grade Kubernetes Architecture: MERN Chat Application

This repository contains the complete Kubernetes (K8s) infrastructure and deployment configurations for a Full-Stack Real-Time Chat Application. The primary focus of this project is to design a highly decoupled, scalable, and secure microservices architecture using Kubernetes best practices.

---

## 🛠️ Application Stack (The Workload)
The core application is built using the MERN stack, containerized via Docker:
* **Frontend:** React.js (Served via Nginx)
* **Backend:** Node.js / Express.js (REST API)
* **Database:** MongoDB

---

## 🏛️ Kubernetes Architecture & Engineering Highlights

This deployment moves away from monolithic structures and embraces strict workload isolation, persistent storage, and advanced traffic routing.

### 1. Workload Isolation (Taints, Tolerations & Affinity)
To ensure optimal performance and prevent resource starvation, the workloads are strictly pinned to specific worker nodes:
* **Database Node (Worker 1):** Dedicated solely to MongoDB. Secured using **Node Taints** (`app=database:NoSchedule`) to repel regular pods. The MongoDB pod uses a matching **Toleration** and **Node Affinity** to schedule here.
* **Application Node (Worker 2):** Dedicated to Stateless apps (Node.js Backend & React Frontend) using strict **Node Affinity**.

### 2. Stateful Storage Management
* **StatefulSet:** MongoDB is deployed as a `StatefulSet` (rather than a Deployment) to maintain a sticky identity and stable hostname (`mongodb-0`).
* **Persistent Volume Claim (PVC):** Ensures chat data survives pod restarts and crashes by dynamically provisioning persistent storage mounted at `/data/db`.

### 3. Configuration & Security
* **Kubernetes Secrets:** Sensitive data like MongoDB URIs, Root Passwords, and JWT Secrets are base64 encoded and securely injected into the backend pods as Environment Variables.
* **ConfigMaps (The Monolith Hack):** A `ConfigMap` is creatively used to mount a dummy `index.html` file directly into the backend container's `/frontend/dist/` directory, resolving legacy monolith path dependencies (`ENOENT` errors) without rebuilding Docker images.

### 4. Networking & Traffic Routing
* **ClusterIP Services:** Used for internal, secure communication (e.g., Backend communicating with MongoDB).
* **Nginx Ingress Controller:** Acts as the cluster gateway. An `Ingress` resource is configured as a catch-all (`rewrite-target: /`) to elegantly route external Port 80 traffic directly to the Frontend service, eliminating the need for exposed NodePorts in production.

### 5. Resilience & Resource Allocation
* **Resource Quotas:** Every container (Frontend, Backend, DB) has strict `limits` and `requests` for CPU and Memory to prevent memory leaks from crashing the nodes.
* **Probes:** Configured readiness and liveness checks to ensure traffic is only sent to healthy pods, triggering automatic restarts (`CrashLoopBackOff` mitigation) when the app hangs.

---

## 🚀 How to Deploy

**Prerequisites:**
* A running Kubernetes Cluster (e.g., KIND, Minikube, EKS, or EC2 hosted).
* `kubectl` installed and configured.
* Nginx Ingress Controller installed on the cluster.

### **Deployment Commands:**

Follow this sequence to ensure dependencies are met:

**1. Apply the Secret:** (Ensure to add your base64 encoded credentials first)
```bash
kubectl apply -f k8s/01-mongo-secret.yml
2. Deploy the Database Layer:

Bash
kubectl apply -f k8s/02-mongo-pvc.yml
kubectl apply -f k8s/03-mongo-statefulset.yml
kubectl apply -f k8s/04-mongo-svc.yml
3. Deploy the Configuration Hack:

Bash
kubectl apply -f k8s/10-configmap.yml
4. Deploy the Application Layer (Backend & Frontend):

Bash
kubectl apply -f k8s/05-backend-deployment.yml
kubectl apply -f k8s/06-backend-svc.yml
kubectl apply -f k8s/07-frontend-deployment.yml
kubectl apply -f k8s/08-frontend-svc.yml
5. Apply the Ingress Rules:

Bash
kubectl apply -f k8s/09-ingress.yml
🔍 Verification
To check if all components are running correctly:

Bash
# Check Pods and Nodes
kubectl get pods -o wide
kubectl get nodes --show-labels

# Check Ingress
kubectl get ingress
👨‍💻 Author
Wajahat Rasool DevOps & Cloud Computing Specialist