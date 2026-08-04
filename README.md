# On-Demand Kubernetes Lab

A lightweight Kubernetes practice lab using **k3d + K3s + Docker** on Ubuntu.

## 1. Clone the Repository

```bash
git clone https://github.com/nawab06/k8s-on-demand-lab.git
cd k8s-on-demand-lab
```

## 2. Install Prerequisites

Update packages and install required tools:

```bash
sudo apt update
sudo apt install -y curl ca-certificates
```

### Install k3d

```bash
curl -fsSL https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

### Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

rm -f kubectl
```

Verify:

```bash
docker --version
k3d version
kubectl version --client
```

## 3. Configure Docker Access

Your user must be able to access Docker without `sudo`.

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

Verify:

```bash
docker ps
```

If `docker ps` works without `sudo`, continue.

## 4. Install the Lab Commands

From the cloned repository:

```bash
sudo ln -sf "$(pwd)/k8s-start" /usr/local/bin/k8s-start
sudo ln -sf "$(pwd)/k8s-stop" /usr/local/bin/k8s-stop
sudo ln -sf "$(pwd)/k8s-status" /usr/local/bin/k8s-status
sudo ln -sf "$(pwd)/k8s-destroy" /usr/local/bin/k8s-destroy
```

Make sure the scripts are executable:

```bash
chmod +x k8s-start k8s-stop k8s-status k8s-destroy
```

## 5. Start the Kubernetes Lab

```bash
k8s-start
```

The first execution creates the Kubernetes cluster.

Check the nodes:

```bash
kubectl get nodes
```

All three nodes should show `Ready`.

## 6. Check Lab Status

```bash
k8s-status
```

This shows the Kubernetes containers and node status.

## 7. Stop the Lab

When you finish practicing:

```bash
k8s-stop
```

This stops the Kubernetes containers without deleting the cluster.

The Ubuntu VM and Docker daemon can continue running for other work.

## 8. Start the Lab Again

When you want to practice Kubernetes again:

```bash
k8s-start
```

The existing cluster will be started again.

## 9. Delete the Lab

To permanently delete the Kubernetes cluster:

```bash
k8s-destroy
```

You will be asked to confirm the deletion.

Use this only when you want to completely rebuild the lab.

## Typical Workflow

```bash
# Start Kubernetes
k8s-start

# Check nodes
kubectl get nodes

# Practice Kubernetes
kubectl get pods -A

# Check lab status
k8s-status

# Finish the lab
k8s-stop
```

## On-Demand Design

The Kubernetes cluster is intentionally **on-demand**.

```text
Ubuntu VM
   |
   +-- Docker daemon       → Running
   |
   +-- Kubernetes cluster  → OFF by default
                              |
                         k8s-start
                              ↓
                         Kubernetes ON
                              |
                         Practice
                              |
                         k8s-stop
                              ↓
                         Kubernetes OFF
```

The lab does not configure Kubernetes to automatically start when the Ubuntu VM boots.
