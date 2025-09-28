# 🚀 Kubernetes Demo App Labs
This repository contains a simple demo Node.js application to help you learn how to deploy apps using Kubernetes locally via Minikube.

## 🛠️ Prerequisites
#### ✅ Install Minikube
**Minikube** helps you run Kubernetes locally. It creates a single-node Kubernetes cluster inside a virtual machine to simulate a cloud environment.

Download link: https://minikube.sigs.k8s.io/docs/

After installation, start the dashboard with:

```bash
minikube start
minikube status
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
docker build -t first-node .
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
docker tag <image_name> <your-dockerhub-username>/<repo_name>:<version>
docker tag first-node jrojascr/kubernetes-demo:latest
docker tag first-node jrojascr/kubernetes-demo:v1 # specific version tag
```
d. Push the image:
```bash
docker push <your-dockerhub-username>/<repo_name>
docker push jrojascr/kubernetes-demo
```

## ☸️ Deploying the App on Kubernetes
Kubernetes can only use images that are available in a container registry (like Docker Hub), so make sure you’ve pushed your image.

You can deploy the app using two approaches: **Imperative** and **Declarative**.

### 🔧 1. Imperative Deployment
a. Create a Deployment</br>
Make sure Minikube is running first by starting it with `minikube start`:
```bash
kubectl create deployment first-app --image=jrojascr/kubernetes-demo:latest
```
b. Verify the deployment and pods:
```bash
kubectl get deployments
kubectl get pods
```
After about 5 seconds, you should see one deployment with `READY 1/1`, indicating that the deployment was successful. You can also verify this via the Minikube dashboard command `minikube dashboard`.

c. Creates a **LoadBalancer service** to expose the **first-app deployment** to traffic on port 8080
```bash
kubectl expose deployment first-app --name=my-service --type=LoadBalancer --port=8080
kubectl expose deployment first-app --type=LoadBalancer --port=8080
```
Now get a message indicating that the deployment was exposed, like this:`service/first-app exposed`

d. Check services:
```bash
kubectl get services # the new service should appear in the list.
```
e. Opens the Kubernetes Service named first-app:
```bash
minikube service <my-service>
minikube service first-app # This will open the app URL in your browser.
```

---
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

###  📄 2. Declarative Deployment (Alternative Approach)
In this approach, we define application resources using YAML configuration files. The advantage of using configuration files is that they enable versioning, automation, and help maintain consistency across different environments.

#### 🧱 Kubernetes Deployment Configuration File
**File:** `deployment.yaml` </br>
This file defines the desired state of a Deployment, including the number of pod replicas, the container image to use, and the pod labels.
```bash
# deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: second-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: second-app
      tier: backend
  template:
    metadata:
      labels:
        app: second-app
        tier: backend
    spec:
      containers:
        - name: second-node
          image: academind/kub-first-app:2
```

#### 🌐 Kubernetes Service Configuration File
**File:** `service.yaml` </br>
This file defines a Service to expose the app defined in the deployment above. It maps incoming traffic to the correct Pods based on their labels.

```bash
# service.yaml

apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    group: example
spec:
  selector:
    app: second-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

### 🚀 Applying the Configuration Files

Make sure Minikube is running first by starting it with `minikube start`:
#### ✅ 1. Apply the Deployment:
a. Create the service with the following:
```bash
kubectl apply -f deployment.yaml
```
Check that the deployment was created:
```bash
kubectl get deployments
kubectl get pods
```
#### ✅ 2. Apply the Service:
```bash
kubectl apply -f service.yaml
```
Verify the service:
```bash
kubectl get services
```
### 🌐 Accessing the App
If you're using Minikube, expose the service in your local environment:
```bash
minikube service backend
```
This will open the app in your default browser using the Minikube service tunnel.

### ✅ Overview of Completed Work
-  Declarative deployments use version-controlled YAML files for consistent and reusable configurations.
- `kubectl apply` simplifies the management and updates of Kubernetes resources.
- Minikube simulates a real-world Kubernetes environment locally for testing and development.
- Services enable reliable access to your Pods, both internally within the cluster and externally when needed.

## ✏️ Docker Useful Commands

See the logs
```bash
docker logs <container_name_or_id>
```
⚠️ WARNING

**Be careful when running these commands! They can delete containers, images, and more.**</br>

1. Stop and remove all containers
```bash
docker stop $(docker ps -q) && docker rm $(docker ps -aq)
```

3. Remove single container
```bash
docker rm <container_name_or_id>
```

4. Remove all images 
```bash
docker rmi $(docker images -q)
```

5. Remove single image
```bash
docker rmi <container_name_or_id>
```
6. Remove volume
```bash
docker volume rm <volume_name_or_id>
```

🔥 Optional full cleanup:
If you're just cleaning up everything (containers, images, networks, build cache):
```bash
docker system prune -a
```
⚠️ This will delete:
All **stopped** containers
All images not used by running containers
All networks not used
All build cache
You’ll be prompted to confirm before it runs.

🔥 Delete All Resources in the default Namespace
```bash
kubectl delete all --all
kubectl delete all --all -n my-namespace # or specific workspace
```
This deletes:
Pods
Deployments
ReplicaSets
Services
DaemonSets
StatefulSets
Jobs

🧯 To Completely Reset Minikube (Optional)
If you're just testing and want to reset everything, including the cluster:
```bash
minikube delete
minikube start
```

