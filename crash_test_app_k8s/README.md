### `README.md`

```markdown
# Streamlit Kubernetes Deployment with Rolling Updates

This project demonstrates how to deploy a Python Streamlit application to a Kubernetes cluster and perform a **rolling update**. The application crashes after handling one request to test pod replication, load balancing, and failover behavior in Kubernetes.

## Prerequisites

- Docker installed on your local machine
- Kubernetes cluster (e.g., Minikube, Docker Desktop Kubernetes, or a managed Kubernetes service like EKS, GKE, or AKS)
- `kubectl` installed and configured to manage the Kubernetes cluster
- Docker Hub or another container registry to push your Docker image

## Steps to Deploy and Update

### 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-folder>
```

### 2. Build and Push Docker Image

After making modifications to the `app.py` file (which crashes after one request), you need to build and push the Docker image.

1. Build the Docker image:

   ```bash
   docker build -t your-docker-username/python-streamlit-app:v2 .
   ```

2. Push the Docker image to Docker Hub (or your preferred registry):

   ```bash
   docker push your-docker-username/python-streamlit-app:v2
   ```

### 3. Create Kubernetes Deployment

Modify the `streamlit-app-deployment.yaml` file to use the new image:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: streamlit-app
spec:
  replicas: 2  # Number of replicas for load balancing
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: streamlit-app
  template:
    metadata:
      labels:
        app: streamlit-app
    spec:
      containers:
      - name: streamlit-app
        image: your-docker-username/python-streamlit-app:v2  # Update image to new version
        ports:
        - containerPort: 8501

---
apiVersion: v1
kind: Service
metadata:
  name: streamlit-service
spec:
  selector:
    app: streamlit-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8501
    nodePort: 30007  # NodePort to access the app externally
  type: NodePort
```

Apply the deployment and service to the Kubernetes cluster:

```bash
kubectl apply -f streamlit-app-deployment.yaml
```

### 4. Monitor Rolling Update

You can monitor the status of the rolling update using the following command:

```bash
kubectl rollout status deployment/streamlit-app
```

During the rolling update, Kubernetes will replace old pods with new ones without any downtime.

To see the new pods being created and the old ones being terminated:

```bash
kubectl get pods -w
```

### 5. Access the App

Once the pods are running, you can access the Streamlit app through your NodePort service. In your browser, go to:

```
http://<NODE-IP>:30007
```

Here, `<NODE-IP>` is the IP address of any node in your Kubernetes cluster.

### 6. Simulate Pod Failover

The application is designed to crash after handling one request. Click the **"Start Heavy Load"** button to make the app crash, and Kubernetes will automatically restart the pod.

You can simulate multiple users accessing the app and observe how Kubernetes handles pod failover and load balancing.

### 7. Roll Back Deployment (if needed)

If you need to roll back to a previous version of the deployment, use the following command:

```bash
kubectl rollout undo deployment/streamlit-app
```

### Additional Commands

- **Check the rollout history**:
  
  ```bash
  kubectl rollout history deployment/streamlit-app
  ```

- **View pod logs**:

  ```bash
  kubectl logs <pod-name>
  ```

- **Delete the deployment and service**:

  To clean up the resources created by this deployment:

  ```bash
  kubectl delete deployment streamlit-app
  kubectl delete service streamlit-service
  ```

## Application Overview

The application is a basic Python Streamlit app that simulates heavy computation and crashes after processing one request. The primary purpose is to test the **replication**, **load balancing**, and **failover** features of Kubernetes.

### Key Features:

- **Streamlit Application**: A simple web app with a button that triggers heavy computation and crashes the app after handling the request.
- **Kubernetes Deployment**: The app is deployed on Kubernetes using a rolling update strategy.
- **NodePort Service**: The service allows external access to the application using a NodePort.
- **Rolling Updates**: Kubernetes rolling updates ensure zero downtime while updating the application.
