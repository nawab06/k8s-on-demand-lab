On-Demand Kubernetes Lab with k3d

A lightweight, reusable Kubernetes practice environment running a 3-node K3s cluster inside a single Ubuntu VM using Docker and k3d.

The main design goal is on-demand operation: Kubernetes containers do not need to run when the VM is being used for other labs. Start the cluster only when Kubernetes practice is required, and stop it afterward to release CPU and RAM.

Architecture

Ubuntu 22.04 VM
├── Docker daemon
│
└── k3d
    └── k8s-lab
        ├── k3d-k8s-lab-server-0   # Control plane
        ├── k3d-k8s-lab-agent-0    # Worker
        └── k3d-k8s-lab-agent-1    # Worker

Host resources used for this lab

Resource

Configuration

OS

Ubuntu 22.04.5 LTS

CPU

8 vCPU

RAM

7.5 GiB

Swap

2 GiB

Disk

49 GiB

Free disk at setup

~33 GiB

Container runtime

Docker 29.7.1

Kubernetes distribution

K3s

k3d

v5.9.0

kubectl

v1.36.3

K3s version

v1.35.5-k3s1

Versions above document the environment used to create this lab. They may differ if the installation is performed later.

Why k3d?

k3d runs lightweight K3s Kubernetes nodes as Docker containers. It is a good fit for a personal laptop/VM lab because it provides a multi-node Kubernetes environment without requiring multiple full virtual machines.

This lab intentionally uses:

1 Kubernetes control-plane node

2 Kubernetes worker nodes

K3s

containerd inside K3s

Docker on the Ubuntu host

Traefik disabled so ingress can be introduced later as a separate learning exercise

Prerequisites

The Ubuntu VM should have:

Ubuntu 22.04 or a compatible Linux distribution

Docker installed and running

A user with permission to access /var/run/docker.sock

curl

k3d

kubectl

For Docker access without sudo, add the current user to the Docker group:

sudo usermod -aG docker "$USER"
newgrp docker
docker ps

Membership in the Docker group effectively grants root-level control over the host. Use this only on a trusted personal/lab system.

Installation

Install Docker using the official Docker installation instructions for your Linux distribution.

Install k3d:

curl -fsSL https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

Install the latest stable kubectl:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm -f kubectl

Verify:

k3d version
kubectl version --client
docker --version

Repository layout

k8s-lab/
├── k8s-start
├── k8s-stop
├── k8s-status
├── k8s-destroy
└── README.md

The scripts can be kept in ~/k8s-lab/ on the VM.

Commands

Start the lab

k8s-start

The first execution creates the cluster. Later executions start the existing Kubernetes containers and wait for the API server and all three nodes to become ready.

Expected result:

NAME                   STATUS   ROLES
k3d-k8s-lab-agent-0    Ready    <none>
k3d-k8s-lab-agent-1    Ready    <none>
k3d-k8s-lab-server-0   Ready    control-plane

Check status

k8s-status

This displays the Kubernetes containers and, when the cluster is running, the Kubernetes nodes.

Stop the lab

k8s-stop

This stops the k3d Kubernetes containers without deleting the cluster.

The cluster data/configuration remains available so it can be started again later.

Destroy the lab

k8s-destroy

This permanently deletes the k8s-lab k3d cluster after confirmation.

Use this only when you intentionally want to rebuild the environment.

On-demand lifecycle

The intended workflow is:

VM boot
   |
   v
Docker daemon running
   |
   +---- Other labs/work
   |
   v
k8s-start
   |
   v
3-node Kubernetes cluster
   |
   +---- Kubernetes practice
   |
   v
k8s-stop
   |
   v
Kubernetes containers stopped
   |
   +---- Resources available for other labs
   |
   v
k8s-start
   |
   v
Cluster available again

The Kubernetes cluster is not configured as a systemd service and is not intended to start automatically when Ubuntu boots.

Docker itself may remain enabled because other Docker-based labs may depend on it.

Kubernetes exercises planned for this lab

This environment is intended for hands-on practice with:

Fundamentals

Kubernetes architecture

kubectl

Namespaces

Pods

Labels and selectors

Annotations

Workloads

Deployments

ReplicaSets

Scaling

Rolling updates

Rollbacks

Jobs

CronJobs

DaemonSets

StatefulSets

Networking

ClusterIP Services

NodePort Services

LoadBalancer Services

DNS

Ingress

NetworkPolicies

Configuration and storage

ConfigMaps

Secrets

EmptyDir

PersistentVolumes

PersistentVolumeClaims

StorageClasses

Scheduling

Node labels

Node selectors

Affinity/anti-affinity

Taints and tolerations

Resource requests and limits

Security

ServiceAccounts

RBAC

Roles

ClusterRoles

RoleBindings

Security contexts

Operations and troubleshooting

Pod failures

CrashLoopBackOff

ImagePullBackOff

Pending pods

Scheduling failures

Service connectivity

DNS troubleshooting

Logs

Events

Resource troubleshooting

Package management and observability

Helm

Prometheus

Grafana

Kubernetes metrics

Cluster troubleshooting

Useful verification commands

Check nodes:

kubectl get nodes -o wide

Check all namespaces:

kubectl get pods -A

Check cluster information:

kubectl cluster-info

Check Kubernetes contexts:

kubectl config get-contexts

Check current context:

kubectl config current-context

Check cluster resources:

kubectl get all -A

Troubleshooting

Docker permission denied

If k3d reports:

permission denied while trying to connect to the Docker daemon socket

run:

sudo usermod -aG docker "$USER"
newgrp docker
docker ps

Then retry:

k8s-start

API server temporarily unavailable

When starting a stopped cluster, the Kubernetes API server can take a few seconds to become ready.

The k8s-start script waits for:

The Kubernetes API server readiness endpoint.

All three Kubernetes nodes to report Ready.

Check running containers

docker ps

Check all k3d containers

docker ps -a --filter "name=k3d-k8s-lab"

Check k3d cluster

k3d cluster list

Important notes

Docker group security

Adding a user to the docker group provides broad control over the Docker daemon and therefore effectively root-equivalent capabilities on the host. This is acceptable for a trusted personal lab but should be considered carefully on shared or production systems.

Resource usage

When the Kubernetes lab is stopped with:

k8s-stop

the Kubernetes node containers are stopped. Docker itself remains running because it is used by the host and potentially by other labs.

Persistence

Stopping the cluster is not the same as deleting it.

Use:

k8s-stop

to stop it temporarily.

Use:

k8s-destroy

only when you want to delete the cluster and rebuild it.
