# Flask + MongoDB on Kubernetes (Minikube)

This project demonstrates deploying a **Flask application with MongoDB** on a **Kubernetes cluster using Minikube**, including **persistent storage**, **service-based DNS communication**, and **Horizontal Pod Autoscaling (HPA)**.

The setup is designed for **local development and learning purposes**, focusing on real-world Kubernetes concepts like Deployments, StatefulSets, Services, Secrets, Persistent Volumes, and Autoscaling.

---

## 📌 Project Overview

* **Flask Application**: Python-based REST service
* **MongoDB**: Database deployed using StatefulSet with persistent storage
* **Kubernetes Resources**:

  * Deployment
  * StatefulSet
  * Services (ClusterIP & Headless)
  * Secrets
  * PersistentVolumeClaim (PVC)
  * Horizontal Pod Autoscaler (HPA)
* **Platform**: Minikube (local Kubernetes cluster)

---

## 🧱 Architecture

```
User
 ↓
NodePort Service
 ↓
Flask Pods (Deployment + HPA)
 ↓  (DNS: mongodb)
MongoDB Service (Headless)
 ↓
MongoDB Pod (StatefulSet + PVC)
```

---

## 🐳 Dockerfile (Flask Application)

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

ENV FLASK_APP=app.py
ENV FLASK_ENV=production

CMD ["flask", "run", "--host=0.0.0.0"]
```

---

## 🐳 Build & Push Docker Image

### Build Image (Local)

```bash
docker build -t flask-mongo-app:1.0 .
```

### Tag Image

```bash
docker tag flask-mongo-app:1.0 <docker-username>/flask-mongo-app:1.0
```

### Push to Docker Hub

```bash
docker push <docker-username>/flask-mongo-app:1.0
```

> Note: When using Minikube, the image can also be built directly inside Minikube using:

```bash
minikube docker-env | Invoke-Expression
```

---

## ☸️ Kubernetes YAML Files

### 1️⃣ MongoDB Secret (`mongo-secret.yaml`)

Stores MongoDB credentials securely.

### 2️⃣ MongoDB StatefulSet (`mongo-statefulset.yaml`)

* Ensures stable network identity
* Uses PersistentVolumeClaim for data persistence

### 3️⃣ MongoDB Service (`mongo-service.yaml`)

* Headless service (`ClusterIP: None`)
* Enables DNS-based pod discovery

### 4️⃣ Flask Deployment (`flask-deployment.yaml`)

* Runs multiple replicas
* Uses environment variables to connect to MongoDB

### 5️⃣ Flask Service (`flask-service.yaml`)

* Exposes Flask app using NodePort

### 6️⃣ Horizontal Pod Autoscaler (`flask-hpa.yaml`)

* Scales Flask pods based on CPU usage

---

## 🚀 Deployment Steps (Minikube)

### 1️⃣ Start Minikube

```bash
minikube start
```

### 2️⃣ Apply MongoDB Resources

```bash
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo-statefulset.yaml
kubectl apply -f mongo-service.yaml
```

### 3️⃣ Apply Flask Resources

```bash
kubectl apply -f flask-deployment.yaml
kubectl apply -f flask-service.yaml
```

### 4️⃣ Enable Metrics Server (Required for HPA)

```bash
minikube addons enable metrics-server
```

### 5️⃣ Apply HPA

```bash
kubectl apply -f flask-hpa.yaml
```

### 6️⃣ Verify Deployment

```bash
kubectl get pods
kubectl get svc
kubectl get hpa
kubectl top pods
```

### 7️⃣ Access Application

```bash
minikube service flask-app
```

---

## 🌐 DNS Resolution in Kubernetes

Kubernetes provides **built-in DNS** using **CoreDNS**.

* Each Service gets a DNS name
* Pods can communicate using service names

### Example:

Flask connects to MongoDB using:

```
mongodb:27017
```

Resolution flow:

```
mongodb → mongodb.default.svc.cluster.local → ClusterIP → MongoDB Pod
```

This allows **service-to-service communication without hardcoding IPs**.

---

## 📊 Resource Requests & Limits

Example (Flask Deployment):

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

### Why This Matters:

* **Requests**: Used by scheduler & HPA
* **Limits**: Prevent resource overuse
* HPA calculates scaling based on **CPU requests**, not limits

---

## 🧠 Design Choices

### Flask as Deployment

* Stateless application
* Easy horizontal scaling

### MongoDB as StatefulSet

* Requires stable storage
* Persistent identity
* Ordered startup

### Headless Service for MongoDB

* Enables direct pod addressing
* Required for StatefulSets

### ClusterIP for Internal Communication

* Secure
* Internal-only traffic

### HPA Based on CPU

* Simple and effective
* Supported natively by Kubernetes

### Minikube

* Lightweight
* Ideal for local development and learning

---

## 🧪 Testing & Autoscaling Scenarios

### Load Testing

A BusyBox pod was used to simulate traffic:

```bash
kubectl run load-generator --image=busybox --restart=Never -- sleep 3600
kubectl exec -it load-generator -- sh

while true; do
  wget -q -O- http://flask-app:5000/ > /dev/null
done
```

### Observations

* CPU usage increased
* HPA monitored metrics
* Flask replicas maintained minimum pods

### Issues Encountered

* Metrics-server delay after startup
* PowerShell syntax differences on Windows

All issues were resolved successfully.

---

## 🧹 Cleanup

```bash
kubectl delete deployment flask-app
kubectl delete statefulset mongodb
kubectl delete svc flask-app mongodb
kubectl delete pvc --all
minikube stop
```

---

## ✅ Final Status

✔ Flask app running
✔ MongoDB persistent storage working
✔ DNS-based inter-pod communication
✔ Autoscaling configured and tested
✔ Metrics server operational

---

## 📎 Conclusion

This project successfully demonstrates a **production-style Kubernetes deployment** using Minikube, covering core Kubernetes concepts such as scalability, persistence, service discovery, and containerization.

---

✨ **End of README**
