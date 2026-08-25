# 🍔 Cloud-Native Food Delivery Platform

A **cloud-native food delivery application** demonstrating modern DevOps practices including **Docker, Kubernetes, Terraform, and Jenkins CI/CD**.

## 🏗️ Architecture

```text
Developer
    │
    ▼
 GitHub
    │
    ▼
 Jenkins CI/CD
    │
    ├── Build
    ├── Docker Image
    └── Deploy
          │
          ▼
     Kubernetes
          │
       ┌──┴──┐
      Pod   Pod
       │     │
       └──┬──┘
          ▼
       Service
```

## 🛠️ Tech Stack

* **Docker** – Containerization
* **Kubernetes** – Container orchestration
* **Terraform** – Infrastructure as Code
* **Jenkins** – CI/CD automation
* **GitHub** – Version control
* **JavaScript** – Application layer

## 🚀 Key Features

* Containerized application using Docker
* Kubernetes-based deployment and service management
* Infrastructure provisioning using Terraform
* Automated CI/CD pipeline using Jenkins
* Declarative and reproducible deployment workflow

## 🔄 Deployment Flow

```text
Git Push → Jenkins → Build → Docker Image → Registry → Kubernetes → Application
```

## ▶️ Run Locally

```bash
git clone https://github.com/Kunal-Katkar/Cloud-Native-Food-Delivery-Platform.git
cd Cloud-Native-Food-Delivery-Platform
npm install
npm start
```

### Docker

```bash
docker build -t food-delivery-app .
docker run -p 3000:3000 food-delivery-app
```

### Kubernetes

```bash
kubectl apply -f Kubernetes/
kubectl get pods
kubectl get services
```

## 🎯 Objective

The project demonstrates how a web application can be **containerized, provisioned, automated, and deployed using a cloud-native DevOps workflow**.
