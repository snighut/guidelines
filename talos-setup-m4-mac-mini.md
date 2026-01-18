"Why" and "How" to install Talos OS, tailored for my M4 Mac Mini and GMKtec K8 Plus Mini PC (96GB RAM and 2 TB SSD).

---

# 🚀 Production-Ready AI Homelab

> **Infrastructure-as-Code with Talos Linux, Kubernetes, and Apple Silicon**

This repository contains the configuration, manifests, and documentation for my hybrid homelab. The goal is to mimic a real-world production environment using an immutable operating system for services and a high-performance development station for AI orchestration.

---

## 🏗️ Hardware Architecture

| Device | Role | Specs | OS / Hypervisor |
| --- | --- | --- | --- |
| **GMKtec K8 Plus** | Production Cluster | Ryzen 7 8845HS, 96GB RAM, 2TB SSD | Proxmox + Talos Linux |
| **M4 Mac Mini** | Dev & AI Management | Apple M4 Chip, 16GB+ RAM | macOS (Apple Silicon) |

---

## 🛠️ Why Talos Linux?

In this setup, I use **Talos Linux** for the Kubernetes nodes. Unlike traditional Linux distributions (Ubuntu, Debian), Talos is:

* **Immutable:** The file system is read-only. You cannot manually break the OS by changing files.
* **API-Driven:** There is **no SSH**. All management is done via the `talosctl` API from my Mac. This eliminates "configuration drift."
* **Minimal:** It contains only the bare essentials to run Kubernetes (~12 binaries). This makes it faster and significantly more secure (smaller attack surface).
* **Cattle, Not Pets:** If a node misbehaves, I don't "fix" it; I reset it and apply the config again in seconds.

---

## 💻 M4 Mac Mini: Setup Guide

The M4 Mac Mini acts as the "Command Center." It is used for writing code, building container images, and managing the cluster.

### 1. Install Management Tools

Since the M4 uses Apple Silicon (ARM64), use the official Sidero Labs tap to get the native binaries:

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Talos Control & Kubernetes CLI
brew tap siderolabs/tap
brew install siderolabs/tap/talosctl
brew install kubectl

```

### 2. Configure Local Environment

To manage the cluster securely, define your configuration path in your shell:

```bash
# Add to your .zshrc
export TALOSCONFIG=$HOME/your-repo-path/talosconfig
export KUBECONFIG=$HOME/your-repo-path/kubeconfig

```

---

## ☸️ Cluster Initialization

### 1. Generate Secrets

Run this on the Mac to create the cluster's "Birth Certificate":

```bash
talosctl gen config "homelab-cluster" https://<GMKTEC_VM_IP>:6443

```

### 2. Apply to Proxmox VM

Once the Talos VM is booted in Proxmox, apply the generated machine configuration:

```bash
talosctl apply-config --insecure --nodes <GMKTEC_VM_IP> --file controlplane.yaml

```

### 3. Bootstrap Kubernetes

```bash
# Initialize the K8s control plane
talosctl bootstrap --nodes <GMKTEC_VM_IP>

# Pull the credentials to your Mac
talosctl kubeconfig .

```

---

## 🤖 AI Orchestration Goals

* **GPU Passthrough:** Passing the Radeon 780M iGPU into the Talos Worker VM.
* **Serving:** Deploying **Ollama** & **LocalAI** for LLM inference.
* **Development:** Using **MLX** on the M4 Mac for local model prototyping.
* **GitOps:** Managing all AI deployments via **ArgoCD** for automated synchronization.

---

### Useful Commands

* `talosctl health`: Check cluster status.
* `talosctl dashboard`: Real-time terminal UI for node monitoring.
* `kubectl get nodes`: Verify Kubernetes is running.

---

**Next Step:** Would you like me to generate a **`.gitignore`** file for this repository to ensure you don't accidentally commit your sensitive `talosconfig` secrets to GitHub?
