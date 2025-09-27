# 🚀 Kubernetes Demo App Labs
This repository contains a simple demo Node.js application to help you learn how to deploy apps using Kubernetes locally via Minikube.

## 🛠️ Prerequisites
#### ✅ Install Minikube
**Minikube** helps you run Kubernetes locally. It creates a single-node Kubernetes cluster inside a virtual machine to simulate a cloud environment.

Download link: https://minikube.sigs.k8s.io/docs/

After installation, start the dashboard with:

```bash
minikube dashboard
```
This opens a web UI where you can monitor your cluster, pods, services, and more.

### ✅ Install kubectl
**kubectl** is the command-line tool for interacting with Kubernetes clusters.
It lets you deploy applications, manage resources, and view logs.

Download link: https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-kubectl-binary-with-curl-on-macos

## 🐳 Check Existing Docker Images and Containers

🔍 View local Docker images and containers:
```bash
# List all local Docker images
docker images

# List all containers (running and stopped)
docker ps -a

```

## 📦 Build and Push the App Image
1. Build the Docker image
```bash
docker build -t goals-node .
```
2. Push to Docker Hub
a. Log in:
```bash
docker login
```
b. Create a repository on Docker Hub
Go to https://hub.docker.com and create a new repo (e.g., goals-node).
c. Tag the image:
```bash
docker tag goals-node <your-dockerhub-username>/goals-node:latest
docker tag goals-node <your-dockerhub-username>/goals-node:v1
```
d. Push the image:
```bash
docker push <your-dockerhub-username>/goals-node:latest
docker push <your-dockerhub-username>/goals-node:v1
```

## ☸️ Deploying the App on Kubernetes
Kubernetes can only use images that are available in a container registry (like Docker Hub), so make sure you’ve pushed your image.

You can deploy the app using two approaches: **Imperative** and **Declarative**.

### 🔧 1. Imperative Deployment
a. Create a Deployment:
```bash
kubectl create deployment first-app --image=<your-dockerhub-username>/goals-node:latest
```
b. Verify the deployment and pods:
```bash
kubectl get deployments
kubectl get pods
```
If you see `READY 1/1`, the app is running.
You can also confirm via the Minikube dashboard.

c. Expose the app with a service:
```bash
kubectl expose deployment first-app --type=LoadBalancer --port=8080
```
d. Check services:
```bash
kubectl get services
```
e. Access the app:
```bash
minikube service first-app
```
This will open the app URL in your browser.

### 🔁 Test Restarting and Scaling
Crash a pod intentionally
Use the dashboard or CLI to delete a pod and observe how Kubernetes restarts it automatically.

### Scale the deployment:
```bash
kubectl scale deployment/first-app --replicas=3
```
Check that new pods were created:
```bash
kubectl get pods
```
Try crashing one of the pods and confirm the app continues running – Kubernetes routes traffic to the remaining healthy pods.</br>
**Note**: *This is manual scaling. For auto-scaling, you'll need to configure Horizontal Pod Autoscaling and typically require metrics support, which is easier to set up on a cloud provider.*

###  📄 2. Declarative Deployment (Alternative approach)
Coming soon: Instructions using `kubectl apply -f <yaml-file>` and YAML configuration files for a declarative setup.