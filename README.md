# DevOps Coding Assessment: Ruby on Rails Application Deployment 🚀

## 🌐 Project Overview

This project showcases a full-fledged CI/CD pipeline that automates the build, test, deployment, and management of a Ruby on Rails application with a PostgreSQL database. It emphasizes the integration of various DevOps tools and techniques to achieve a robust and efficient deployment workflow.

## 🔗 Application - Budget App
Giving you the url of the application repostory where is whole project files its deployment files and application overview and files are stored.

https://github.com/sansugupta/Budget-App

## 🏗️ Architecture Diagram

```
[Git Repo] → [Tekton Pipeline] → [Docker Hub]
    ↓                ↓              ↓
[Build] → [Test] → [Push Image] → [Kubernetes]
    ↓
[ArgoCD] → [Deploy]
```

## 🛠️ Technical Stack

* **Application:** Ruby on Rails
* **Database:** PostgreSQL
* **Containerization:** Docker
* **Orchestration:** Kubernetes (Minikube)
* **GitOps:** ArgoCD
* **CI/CD:** Tekton Pipelines

## 📂 Project Structure

```
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
```

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

**Dockerfile.postgres (PostgreSQL Database)**

```dockerfile
FROM postgres:14

# Set environment variables for database configuration
ENV POSTGRES_USER=myuser
ENV POSTGRES_PASSWORD=mypassword
ENV POSTGRES_DB=mydatabase
```

### Kubernetes Manifests

**my-rails-app.yaml (Rails Deployment)**

```yaml
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
```

**my-postgres-db.yaml (PostgreSQL StatefulSet)**

```yaml
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
```

### 🚀 Deployment Process

#### Docker Containerization

```bash
# Build Rails application image
docker build -t <your-dockerhub-username>/my-rails-app:latest .

# Build PostgreSQL image
docker build -f Dockerfile.postgres -t <your-dockerhub-username>/my-postgres-db:latest .

# Push images to Docker Hub
docker push <your-dockerhub-username>/my-rails-app:latest
docker push <your-dockerhub-username>/my-postgres-db:latest
```

#### Kubernetes Deployment

```bash
# Start Minikube
minikube start

# Enable Ingress
minikube addons enable ingress

# Apply Kubernetes manifests
kubectl apply -f my-rails-app.yaml
kubectl apply -f my-rails-app-service.yaml
kubectl apply -f my-postgres-db.yaml
kubectl apply -f ingress.yaml
```

## 🔒 Security Considerations

* RBAC: Implement Role-Based Access Control in Kubernetes to restrict access to resources
* Network Policies: Define Network Policies to control communication between pods
* Image Security: Use security scanning tools like Trivy to scan Docker images
* Secrets Management: Store sensitive information in Kubernetes Secrets

## 📊 Monitoring and Logging

* **Monitoring:** Integrate Prometheus and Grafana to monitor application performance
* **Logging:** Centralize logs using Fluentd or Elasticsearch

## 📈 Performance Optimization

* Set resource limits for containers
* Configure Horizontal Pod Autoscaler
* Implement caching mechanisms

## 🚧 Key Challenges and Solutions

* Ensuring Rails application database connections
* Configuring persistent storage for PostgreSQL
* Implementing GitOps workflow
* Configuring Tekton pipeline

## 📖 Learning Outcomes

* Docker Containerization
* Kubernetes Deployment
* GitOps Principles
* CI/CD Pipeline Construction

## 🔗 Repository

https://github.com/sansugupta/Ruby-on-Rails-Application-Deployment

## 📬 Contact

* **Name:** Sanskar Gupta
* **Email:** Sanskargupta966@gmail.com
* **LinkedIn:** https://www.linkedin.com/in/sanskargupta9/

---

**Note:** This project is a demonstration of DevOps practices and should be used as a learning resource.
