
# Ultimate Kubernetes GitOps \& Service Mesh Lab

🚀 This repository contains a comprehensive, hands-on lab for building a production-grade application delivery and management platform on Kubernetes. You will go from a simple service to a secure, multi-environment GitOps pipeline with automated progressive delivery.

This project is a perfect sandbox for anyone looking to master modern cloud-native practices.

## ✨ Core Concepts \& Architecture

This lab demonstrates the integration of best-in-class cloud-native tools to achieve a secure, automated, and resilient system.

- **GitOps**: Using Git as the single source of truth for declarative infrastructure and applications. Argo CD continuously reconciles the cluster state with the Git repository.
- **Service Mesh**: Using Istio to control, secure, and observe traffic between microservices without changing application code. This includes mTLS, authorization, and advanced traffic management.
- **Progressive Delivery**: Safely releasing new application versions using canary deployments managed by Argo Rollouts, which intelligently shifts traffic based on success metrics from Prometheus.
- **Policy-as-Code**: Enforcing security and best-practice guardrails at the cluster level using OPA Gatekeeper.
- **Multi-Environment Management**: A declarative "App of Apps" pattern to manage dev, staging, and prod environments from a single Git repository using Kustomize overlays.


### Architectural Overview

The workflow is simple yet powerful:

1. A developer pushes a change to the Git repository.
2. Argo CD detects the change and syncs the manifests.
3. For an application update, an Argo Rollouts object is updated, starting a canary release.
4. Argo Rollouts modifies an Istio VirtualService to gradually shift a small percentage of traffic to the new version.
5. During the rollout, it queries Prometheus to ensure success metrics are met before proceeding.
6. All traffic is secured and managed by the Istio service mesh, with policies enforced by OPA Gatekeeper.

## 🛠️ Requirements

To run this lab, you will need the following tools installed on your local machine:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/): For running a local Kubernetes cluster.
- [kubectl](https://kubernetes.io/docs/tasks/tools/): The Kubernetes command-line tool.
- [Istio CLI (istioctl)](https://www.google.com/search?q=https://istio.io/latest/docs/setup/getting-started/%23download): For installing and managing the Istio service mesh.
- [Kustomize](https://kustomize.io/): For managing Kubernetes manifest customizations.
- [Argo CD CLI (argocd)](https://argo-cd.readthedocs.io/en/stable/cli_installation/): For managing Argo CD.
- [Argo Rollouts kubectl plugin](https://www.google.com/search?q=https://argo-rollouts.readthedocs.io/en/stable/installation/%23cli-installation): For observing and managing rollouts.
- A GitHub Account and a Personal Access Token (PAT) with repo scopes for authentication.


## 🏁 Getting Started: Step-by-Step Guide

### 1. Fork and Clone the Repository

First, fork this repository to your own GitHub account. You will be pushing changes to your own fork. Then, clone it to your local machine:

```
# Replace with your GitHub username
git clone https://github.com/YOUR_USERNAME/k8s-gitops-lab.git
cd k8s-gitops-lab
```


### 2. Start Your Kubernetes Cluster

We'll use Minikube to create a local cluster with sufficient resources.

```
minikube start --memory=8192 --cpus=4 --driver=docker --profile=gitops-lab
```


### 3. Install Istio

Install Istio with a demo profile to get started.

```
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
```


### 4. Install Argo CD \& Argo Rollouts

These controllers will manage our GitOps and progressive delivery workflows.

```
# Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```


### 5. Configure Argo CD to Use Your Repository

This is the final and most important step. You need to tell Argo CD where to find your configuration.

Update the repoURL: Open the apps-root.yaml file in the root of this repository and change the repoURL to point to your fork.

Apply the Root Application: This one command bootstraps the entire environment using the "App of Apps" pattern.

```
kubectl apply -f apps-root.yaml
```


### 6. Access Argo CD and Watch the Magic!

Get the initial Argo CD password:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Forward the port to your local machine (in a new terminal):

```
kubectl -n argocd port-forward svc/argocd-server 8081:443
```

Log in to https://localhost:8081 with username admin and the password you just retrieved. You will see the apps-root application, which will automatically deploy the mesh-lab-dev and mesh-lab-staging applications.

You now have a fully functional, multi-environment GitOps pipeline running on your local machine! You can now experiment by making changes in your Git repository and watching Argo CD automatically apply them to your cluster.


  
<img width="1112" height="285" alt="image" src="https://github.com/user-attachments/assets/1a36bc2b-3fe7-41b3-ba39-8a4c5224d8f9" />

***
