# Kubernetes Web Server Deployment

## Theory Questions (10 points)

### a) Kubernetes Architecture

**Q: What are the core components of a Kubernetes cluster (e.g., master, node, etcd, kube-apiserver)? Briefly explain their roles.**

A: Kubernetes has several key components:
- **Master Node**: Controls the cluster and schedules workloads.
- **Worker Nodes**: Run applications in pods.
- **etcd**: Stores cluster data and configuration.
- **kube-apiserver**: Handles communication between cluster components.

**Q: What is a pod in Kubernetes, and how does it differ from a Docker container?**

A: A pod is the smallest deployable unit in Kubernetes and can contain multiple containers that share networking and storage. Unlike a standalone Docker container, a pod allows multiple containers to work together as a single entity.

### b) Deployments and Services

**Q: Explain the purpose of a Kubernetes deployment. How do deployments ensure high availability of applications?**

A: A deployment manages the lifecycle of pods, ensuring the right number of replicas run at all times. It enables rolling updates, rollback capabilities, and high availability by distributing pods across nodes.

**Q: What are the different types of services in Kubernetes (e.g., ClusterIP, NodePort, LoadBalancer)? When would you use each type?**

A: Kubernetes provides different service types:
- **ClusterIP** (default): Exposes services within the cluster.
- **NodePort**: Makes services accessible via a static port on each node.
- **LoadBalancer**: Uses cloud provider's load balancer to expose services externally.
Use ClusterIP for internal communication, NodePort for basic external access, and LoadBalancer for production-level exposure.

### c) Scaling and Autoscaling

**Q: How does Kubernetes handle scaling? Explain the concept of Horizontal Pod Autoscaler and how it responds to workload changes.**

A: Kubernetes scales workloads by adding or removing pod replicas. The **Horizontal Pod Autoscaler (HPA)** automatically scales pods based on CPU or memory usage. If CPU usage exceeds a defined threshold, HPA increases the replica count, ensuring optimal performance.

---

## Practical Implementation

### Overview
This project demonstrates how to deploy a web server in Kubernetes using a Deployment, expose it internally with a ClusterIP service, expose it externally with a NodePort service, and configure Horizontal Pod Autoscaling (HPA) to scale based on CPU utilization.

### Prerequisites
Ensure you have the following tools installed:
- **Kubernetes Cluster** (Minikube, Kind, or a cloud-based cluster)
- **kubectl** (Kubernetes CLI)
- **A containerized web server image** (e.g., Nginx, Apache)

### Deployment

#### 1. Create a Deployment
Create a YAML file `web-server-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      containers:
      - name: web-server
        image: abrar2510/angularapp:frontend-latest
        ports:
        - containerPort: 80
```

Apply the Deployment:
```bash
kubectl apply -f web-server-deployment.yaml
```

#### 2. Create a ClusterIP Service
Create a YAML file `web-server-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-server-service
spec:
  selector:
    app: web-server
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

Apply the Service:
```bash
kubectl apply -f web-server-service.yaml
```

#### 3. Verify Load Balancing
Run the following command to check the service endpoints:
```bash
kubectl get endpoints web-server-service
```
The output should show multiple pod IPs, indicating proper load balancing.

---

### Service Exposure

#### 4. Expose Deployment using NodePort
Create a YAML file `web-server-nodeport.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-server-nodeport
spec:
  selector:
    app: web-server
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```

Apply the NodePort Service:
```bash
kubectl apply -f web-server-nodeport.yaml
```

#### 5. Access the Service
Find the external IP of a node:
```bash
kubectl get nodes -o wide
```
Then, open a browser and access:
```
http://<node-ip>:30080
```

---

### Autoscaling

#### 6. Set Up Horizontal Pod Autoscaler (HPA)
Enable autoscaling for the deployment:
```bash
kubectl autoscale deployment web-server-deployment --cpu-percent=70 --min=3 --max=10
```

#### 7. Simulate High CPU Usage
Run the following command inside a pod:
```bash
kubectl exec -it <pod-name> -- /bin/bash
apt-get update && apt-get install -y stress
stress --cpu 1 --timeout 60s
```

#### 8. Monitor Autoscaling
Check the HPA status:
```bash
kubectl get hpa
```
Check the number of running pods:
```bash
kubectl get pods
```
If CPU usage exceeds 70%, Kubernetes will scale up to 10 replicas.

---

### Cleanup
To remove the resources:
```bash
kubectl delete deployment web-server-deployment
kubectl delete service web-server-service
kubectl delete service web-server-nodeport
kubectl delete hpa web-server-deployment
```
---

### Conclusion
This project demonstrates a complete Kubernetes deployment process, including load balancing, external exposure, and autoscaling based on CPU utilization.