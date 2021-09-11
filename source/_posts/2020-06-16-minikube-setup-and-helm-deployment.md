---
title: Minikube Setup and Helm Deployment
date: 2020-06-16 17:20:00 Z
categories: [Minikube]
tags: [minikube, helm, kubernetes]
cover: /images/fsqs-tips-tricks-notes.png
---

To run a Kubernetes cluster in your local machine and try our Kubernetes capabilities you can use Minikube. In this article, I will explain how to setup k8s command-line tools like `kubectl` `minikube` `helm`.

<!-- more -->

## Setup k8s tools

---

### kubectl

```shell
# Download
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

# Make the kubectl binary executable
chmod +x ./kubectl

# Move the binary in to your PATH
sudo mv ./kubectl /usr/local/bin/kubectl

# Verify command
kubectl version
```

### minikube

```shell
# Download
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Move the binary into your PATH
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify command
minikube version
```

### helm

```shell
# Install
sudo snap install helm --classic

# Verify command
helm version
```

## Helm Deployment

---

### Create a helm project

```shell
helm create my-project
```

### Deploying using helm

```shell
minikube start

kubectl config use-context minikube

helm dependency update my-project

helm upgrade --install -f my-project/values.yaml deployment-name my-project  --force

kubectl get pods
```

### Delete helm deployment

```shell
 helm delete deployment-name
```

## Commands

---

### Useful Minikube commands

```shell
minikube status       # See if Minikube is running
minikube start        # Create and start Minikube
minikube dashboard    # Access the Kubernetes dashboard running within the Minikube cluster
minikube ssh          # Login into the Minikube VM
minikube addons list  # Show the status of the available add-ons
minikube stop         # Stop Minikube
minikube delete       # Delete the Minikube VM
minikube ip           # Show the Minikube IP
```

### Useful Helm commands

```shell
helm init           # Initialize helm
helm create         # Create helm chart
helm install        # Install helm deployment
```

## References
- https://kubernetes.io/docs/setup/learning-environment/minikube/
- https://helm.sh/docs/intro/install/
- https://helm.sh/docs/helm/
- https://kubernetes.io/docs/tasks/tools/install-kubectl/
