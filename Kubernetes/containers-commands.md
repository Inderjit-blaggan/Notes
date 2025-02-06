# Kubernetes Command index
1. [Kind for self hosted cluster](#kind-for-self-hosted-cluster)
2. 
---

# Kind for self hosted cluster


## 🔹 **1. Creating a Kind Cluster**
### **Basic Kind Cluster**
```bash
kind create cluster --name=my-cluster
```
- This creates a default cluster with a single control-plane node.

### **Custom Configuration**
If you need a multi-node setup, use a config file:
```bash
cat <<EOF | kind create cluster --name=my-cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```
This creates a **1 control-plane + 2 worker nodes** cluster.

---

## 🔹 **2. Listing and Getting Cluster Information**
### **List All Kind Clusters**
```bash
kind get clusters
```

### **Get Kubernetes Cluster Information**
```bash
kubectl cluster-info --context kind-my-cluster
kubectl get nodes
```

### **Get Cluster Nodes**
```bash
kubectl get nodes -o wide
```

---

## 🔹 **3. Deleting a Kind Cluster**
### **Delete a Specific Cluster**
```bash
kind delete cluster --name=my-cluster
```

### **Delete All Kind Clusters**
```bash
for cluster in $(kind get clusters); do kind delete cluster --name=$cluster; done
```

---

## 🔹 **4. Exporting Kubernetes Configuration**
If `kubectl` doesn’t work, set the correct context:
```bash
export KUBECONFIG="$(kind get kubeconfig-path --name=my-cluster)"
kubectl config use-context kind-my-cluster
```

---

## 🔹 **5. Managing Nodes**
### **List Nodes**
```bash
kubectl get nodes
```

### **Restart a Node**
```bash
docker restart $(docker ps --filter "name=kind" --format "{{.ID}}")
```

### **Add a Worker Node (After Cluster is Created)**
Kind does not support adding nodes dynamically after creation. You need to recreate the cluster with a new configuration.

---

## 🔹 **6. Interacting with the Kind Cluster**
### **Deploy a Pod**
```bash
kubectl run nginx --image=nginx --port=80
kubectl get pods
```

### **Expose the Pod as a Service**
```bash
kubectl expose pod nginx --type=NodePort --port=80
kubectl get svc
```

---

## 🔹 **7. Load Docker Images into Kind Cluster**
Kind doesn’t pull images from local Docker by default. You must manually load them.

### **Build and Load a Local Docker Image**
```bash
docker build -t my-app:latest .
kind load docker-image my-app:latest --name=my-cluster
```

### **Verify the Image is Available**
```bash
kubectl run my-app --image=my-app:latest --restart=Never
kubectl get pods
```

---

## 🔹 **8. Networking & Port Forwarding**
### **Forward a Port to Access the Cluster from Host**
```bash
kubectl port-forward svc/nginx 8080:80
```
Now, you can access **`http://localhost:8080`**.

### **Check Cluster Networking**
```bash
kubectl get svc -A
kubectl get pods -A -o wide
```

---

## 🔹 **9. Logging & Debugging**
### **View Logs from Kind Cluster Nodes**
```bash
docker logs $(docker ps --filter "name=kind" --format "{{.ID}}")
```

### **View Kubernetes Pod Logs**
```bash
kubectl logs <pod-name>
```

### **Describe a Pod for Debugging**
```bash
kubectl describe pod <pod-name>
```

### **Enter a Running Pod’s Shell**
```bash
kubectl exec -it <pod-name> -- /bin/sh
```

---

## 🔹 **10. Stopping and Restarting the Kind Cluster**
### **Pause the Cluster (Stop Containers)**
```bash
docker pause $(docker ps --filter "name=kind" --format "{{.ID}}")
```

### **Resume Cluster (Unpause Containers)**
```bash
docker unpause $(docker ps --filter "name=kind" --format "{{.ID}}")
```

### **Completely Restart the Kind Cluster**
```bash
kind delete cluster --name=my-cluster
kind create cluster --name=my-cluster
```

