# DevOps Coding Assessment: Ruby on Rails Application Deployment

This repository contains my solution to a DevOps coding assessment focused on deploying a Ruby on Rails application using a comprehensive CI/CD pipeline with Docker, Kubernetes, ArgoCD, and Tekton.

## 🌐 Project Overview

This project showcases a full-fledged CI/CD pipeline that automates the build, test, deployment, and management of a Ruby on Rails application with a PostgreSQL database. It emphasizes the integration of various DevOps tools and techniques to achieve a robust and efficient deployment workflow.

## 🏗️ Architecture Diagram
[Git Repo] → [Tekton Pipeline] → [Docker Hub]
↓                ↓              ↓
[Build] → [Test] → [Push Image] → [Kubernetes]
↓
[ArgoCD] → [Deploy]

## 🛠️ Technical Stack

* **Application:** Ruby on Rails
* **Database:** PostgreSQL
* **Containerization:** Docker
* **Orchestration:** Kubernetes (Minikube)
* **GitOps:** ArgoCD
* **CI/CD:** Tekton Pipelines

## 📂 Project Structure
├── argocd-cm.yaml
├── tekton-dashboard-role.yaml
├── dockerhub-service-account.yaml
├── dockerhub-secret.yaml
├── pipelinerun.yaml
├── my-workspace-pvc.yaml
├── git-credentials.yaml
├── pipeline.yaml
├── tekton-dashboard-rolebinding.yaml
├── config.json
├── ingress.yaml
├── my-postgres-db.yaml
├── my-rails-app-service.yaml
├── my-rails-app.yaml
├── Dockerfile.postgres
└── Dockerfile

## 🔧 Key Configurations

### Dockerfiles

**Dockerfile (Rails Application)**

```dockerfile
FROM ruby:3.1

WORKDIR /app

COPY Gemfile Gemfile.lock ./
RUN bundle install

COPY . .

# Configure database connection
ENV DATABASE_HOST=my-postgres-db
ENV DATABASE_USER=myuser
ENV DATABASE_PASSWORD=mypassword
ENV DATABASE_NAME=mydatabase

# Expose port 3000
EXPOSE 3000

# Start the Rails server
CMD ["rails", "server", "-b", "0.0.0.0"]
```

```Dockerfile.postgres (PostgreSQL Database)

FROM postgres:14

# Set environment variables for database configuration
ENV POSTGRES_USER=myuser
ENV POSTGRES_PASSWORD=mypassword
ENV POSTGRES_DB=mydatabase
```

Kubernetes Manifests
my-rails-app.yaml (Rails Deployment)

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-rails-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-rails-app
  template:
    metadata:
      labels:
        app: my-rails-app
    spec:
      containers:
      - name: my-rails-app
        image: <your-dockerhub-username>/my-rails-app:latest 
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_HOST
          value: my-postgres-db
        - name: DATABASE_USER
          value: myuser
        - name: DATABASE_PASSWORD
          value: mypassword
        - name: DATABASE_NAME
          value: mydatabase

my-postgres-db.yaml (PostgreSQL StatefulSet)

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-postgres-db
spec:
  serviceName: "my-postgres-db"
  replicas: 1
  selector:
    matchLabels:
      app: my-postgres-db
  template:
    metadata:
      labels:
        app: my-postgres-db
    spec:
      containers:
      - name: my-postgres-db
        image: <your-dockerhub-username>/my-postgres-db:latest
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-persistent-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-persistent-storage
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

ingress.yaml (Ingress Configuration)

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-rails-app-ingress
spec:
  rules:
  - host: my-rails-app.local # Replace with your desired hostname
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-rails-app-service
            port:
              number: 3000

ArgoCD Configuration
argocd-cm.yaml (ArgoCD ConfigMap)

apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd-server.argocd.svc.cluster.local # Replace with your ArgoCD server URL
  repo.server: github.com # Replace with your Git provider

argocd-rbac-cm.yaml (ArgoCD RBAC ConfigMap)

apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    g, <your-github-username>, role:admin

Tekton Pipeline Configuration
pipeline.yaml (Tekton Pipeline)

Tekton Pipeline Configuration
pipeline.yaml (Tekton Pipeline)

🚀 Deployment Process
Docker Containerization

# Build Rails application image
docker build -t <your-dockerhub-username>/my-rails-app:latest .

# Build PostgreSQL image
docker build -f Dockerfile.postgres -t <your-dockerhub-username>/my-postgres-db:latest .

# Push images to Docker Hub
docker push <your-dockerhub-username>/my-rails-app:latest
docker push <your-dockerhub-username>/my-postgres-db:latest

Kubernetes Deployment

# Start Minikube
minikube start

# Enable Ingress
minikube addons enable ingress

# Apply Kubernetes manifests
kubectl apply -f my-rails-app.yaml
kubectl apply -f my-rails-app-service.yaml
kubectl apply -f my-postgres-db.yaml
kubectl apply -f ingress.yaml

ArgoCD Setup

# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

# Access ArgoCD UI
kubectl port-forward service/argocd-server -n argocd 8080:443

# Configure ArgoCD to connect to your repository and deploy the application

Tekton Pipeline

# Install Tekton Pipelines
kubectl apply --filename [https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml](https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml)

# Install Tekton Dashboard
kubectl apply --filename [https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml](https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml)

# Apply Tekton pipeline configurations
kubectl apply -f pipeline.yaml -n tekton-pipelines
kubectl apply -f pipelinerun.yaml -n tekton-pipelines

# Access Tekton Dashboard
kubectl port-forward service/tekton-dashboard -n tekton-pipelines 8081:9097

# Manually trigger the pipeline from the Tekton Dashboard

🔒 Security Considerations
RBAC: Implement Role-Based Access Control in Kubernetes to restrict access to resources.
Network Policies: Define Network Policies to control communication between pods in the cluster.
Image Security: Use security scanning tools like Trivy to scan Docker images for vulnerabilities.
Secrets Management: Store sensitive information like database credentials in Kubernetes Secrets.
📊 Monitoring and Logging
Monitoring: Integrate monitoring tools like Prometheus and Grafana to monitor application performance and resource usage.
Logging: Centralize logs using a logging solution like Fluentd or Elasticsearch for easier troubleshooting.
📈 Performance Optimization
Resource Limits: Set resource limits for containers to prevent resource exhaustion.
Horizontal Pod Autoscaler: Configure HPA to automatically scale the application based on CPU or memory usage.
Caching: Implement caching mechanisms to improve application performance.
🚧 Challenges and Solutions
Database Connection: Ensure the Rails application can connect to the PostgreSQL database running in a separate pod.
Persistent Storage: Configure persistent storage for the PostgreSQL database using PersistentVolumeClaims.
GitOps Workflow: Set up ArgoCD to manage deployments and ensure consistency between Git and the cluster.
Tekton Pipeline: Configure the Tekton pipeline to build, push, and deploy the application correctly.
📖 Learning Outcomes
Docker Containerization: Learned to containerize applications and databases using Docker.
Kubernetes Deployment: Gained experience in deploying applications to Kubernetes using various resources like Deployments, Services, and StatefulSets.
GitOps Principles: Understood and implemented GitOps principles using ArgoCD.
CI/CD Pipeline Construction: Learned to build CI/CD pipelines using Tekton.
🔗 Repository
[Your GitHub Repository Link]

🎬 Video Demo
[Link to Project Demonstration Video]

📬 Contact
Name: [Your Name]
Email: [Your Email]
LinkedIn: [Your LinkedIn Profile]
📜 License
[Specify License]
