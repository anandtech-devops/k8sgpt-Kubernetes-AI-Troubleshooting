# K8sGPT Kubernetes AI Troubleshooting

AI-assisted Kubernetes troubleshooting using **K8sGPT, Ollama, and Llama 3.2** on an AWS EC2-based Kubernetes cluster.

## 📌 Project Overview

This project demonstrates how AI can assist a DevOps Engineer in troubleshooting Kubernetes issues.

An intentional Kubernetes image-pull failure was created using an invalid Docker image tag. **K8sGPT** was used to analyze the Kubernetes resource, while **Ollama running Llama 3.2** provided a human-readable explanation of the problem.

The AI-generated recommendation was manually validated before applying the Kubernetes fix.

## 🏗️ Architecture

```text
                         AWS EC2
              ┌──────────────────────────┐
              │                          │
              │   Kubernetes Cluster     │
              │                          │
              │   ┌──────────────────┐   │
              │   │ Broken Pod       │   │
              │   │ ImagePullBackOff │   │
              │   └────────┬─────────┘   │
              │            │             │
              │            ▼             │
              │       ┌─────────┐        │
              │       │ K8sGPT  │        │
              │       └────┬────┘        │
              │            │             │
              │     localhost:11434      │
              └────────────┼─────────────┘
                           │
                    SSH Reverse Tunnel
                           │
                           ▼
              ┌──────────────────────────┐
              │       Windows PC         │
              │                          │
              │       Ollama             │
              │       Llama 3.2          │
              │       :11434             │
              └──────────────────────────┘
```

## 🛠️ Technologies Used

* AWS EC2
* Kubernetes
* kubeadm
* containerd
* Flannel
* kubectl
* K8sGPT
* Ollama
* Llama 3.2
* Docker / Docker Hub
* Linux
* Windows PowerShell
* OpenSSH
* SSH Reverse Tunnel

## ☁️ Infrastructure

### AWS

* Platform: AWS EC2
* OS: Ubuntu
* Kubernetes: v1.34.x
* Container Runtime: containerd
* Network Plugin: Flannel

### Local AI

* Ollama
* Model: `llama3.2:latest`
* Model size: 3.2B parameters
* Quantization: Q4_K_M

The LLM was intentionally kept on the local Windows machine instead of the EC2 instance to avoid running a resource-intensive AI model on the Kubernetes server.

## 🚀 Kubernetes Setup

A single-node Kubernetes cluster was created using kubeadm.

The cluster was configured with:

* containerd
* kubeadm
* kubelet
* kubectl
* Flannel CNI

Cluster verification:

```bash
kubectl get nodes
```

Example:

```text
NAME                   STATUS   ROLES           VERSION
ip-172-31-38-224       Ready    control-plane   v1.34.x
```

## 🤖 K8sGPT Installation

K8sGPT was installed on the EC2 instance.

Version used:

```bash
k8sgpt version
```

Example:

```text
k8sgpt: 0.4.36
```

Available AI providers were checked using:

```bash
k8sgpt auth list
```

Ollama was configured as the active provider.

## 🔗 Ollama Integration

Ollama was running locally on the Windows machine:

```text
127.0.0.1:11434
```

The installed model was:

```text
llama3.2:latest
```

K8sGPT was configured to use Ollama:

```bash
k8sgpt auth add --backend ollama \
  --model llama3.2 \
  --baseurl http://127.0.0.1:11434
```

Ollama was then configured as the default K8sGPT provider:

```bash
k8sgpt auth default --provider ollama
```

Verification:

```bash
k8sgpt auth list
```

Expected:

```text
Default:
> ollama

Active:
> ollama
```

## 🔐 Secure SSH Reverse Tunnel

Ollama was not exposed directly to the internet.

Instead, an encrypted SSH reverse tunnel was created from Windows to the AWS EC2 instance:

```powershell
ssh -i "C:\Anand\food-k8s-key" `
  -N `
  -R 11434:127.0.0.1:11434 `
  ubuntu@<EC2_PUBLIC_IP>
```

This allowed the EC2 instance to access the Windows Ollama service through:

```text
127.0.0.1:11434
```

The connection was verified from EC2:

```bash
curl http://127.0.0.1:11434/api/tags
```

The response confirmed that `llama3.2:latest` was available.

## 🧪 Creating an Intentional Kubernetes Failure

A test namespace was created:

```bash
kubectl create namespace k8sgpt-lab
```

An intentionally broken deployment was created using a non-existent image tag:

```bash
kubectl create deployment broken-app \
  --image=nginx:this-image-does-not-exist \
  -n k8sgpt-lab
```

The Pod entered:

```text
ErrImagePull
ImagePullBackOff
```

Verification:

```bash
kubectl get pods -n k8sgpt-lab
```

Example:

```text
NAME                          READY   STATUS
broken-app-xxxxxxxxxx-xxxxx  0/1     ImagePullBackOff
```

## 🔍 Manual Troubleshooting

The Kubernetes Pod was investigated using:

```bash
kubectl describe pod <pod-name> -n k8sgpt-lab
```

The events showed:

```text
Failed to pull image
nginx:this-image-does-not-exist
```

The image did not exist, causing Kubernetes to repeatedly retry the image pull.

## 🧠 K8sGPT AI-Assisted Troubleshooting

K8sGPT was used to analyze the failing Pod:

```bash
k8sgpt analyze \
  --filter=Pod \
  --namespace=k8sgpt-lab \
  --explain
```

K8sGPT detected:

```text
Back-off pulling image "nginx:this-image-does-not-exist"

ErrImagePull

failed to resolve image:
docker.io/library/nginx:this-image-does-not-exist
```

The Ollama LLM generated a human-readable explanation and suggested checking the image and updating the deployment.

## 🔧 Fix

The actual Kubernetes container name was first verified:

```bash
kubectl get deployment broken-app \
  -n k8sgpt-lab \
  -o jsonpath='{.spec.template.spec.containers[*].name}{"\n"}'
```

The container name was:

```text
nginx
```

The deployment was then updated with a valid image:

```bash
kubectl set image deployment/broken-app \
  nginx=nginx:latest \
  -n k8sgpt-lab
```

## ✅ Verification

The resulting Pod was verified using:

```bash
kubectl get pods -n k8sgpt-lab
```

Final result:

```text
NAME                          READY   STATUS    RESTARTS
broken-app-xxxxxxxxxx-xxxxx  1/1     Running   0
```

The Kubernetes deployment successfully recovered from `ImagePullBackOff`.

## 🔄 Troubleshooting Workflow

```text
Kubernetes Failure
       ↓
kubectl get pods
       ↓
ImagePullBackOff
       ↓
kubectl describe pod
       ↓
K8sGPT analyze
       ↓
AI explanation using Ollama
       ↓
Engineer validates recommendation
       ↓
Correct Kubernetes configuration
       ↓
Rollout verification
       ↓
Pod Running
```

## 🎯 Key Learnings

* Kubernetes Pod troubleshooting
* `ErrImagePull` and `ImagePullBackOff`
* Kubernetes Deployment and container configuration
* K8sGPT AI-assisted troubleshooting
* Ollama local LLM integration
* Llama 3.2
* Secure SSH reverse tunneling
* AWS EC2-based Kubernetes
* Manual validation of AI-generated recommendations
* Using AI as a troubleshooting assistant rather than blindly applying AI-generated changes

## ⚠️ Security Notes

No private keys, AWS credentials, passwords, or sensitive configuration files are included in this repository.

The Ollama API was not exposed publicly. A secure SSH reverse tunnel was used to connect the AWS EC2 environment to the local Ollama instance.

## 📁 Suggested Repository Structure

```text
k8sgpt-kubernetes-ai-troubleshooting/
│
├── README.md
│
├── kubernetes/
│   ├── namespace.yaml
│   └── broken-deployment.yaml
│
├── screenshots/
│   ├── k8sgpt-analysis.png
│   ├── ollama-model.png
│   ├── kubernetes-error.png
│   └── kubernetes-fixed.png
│
└── docs/
    └── troubleshooting.md
```

## 👨‍💻 Author

**Anand Srivastava**

DevOps Engineer | AWS | Kubernetes | Docker | Jenkins | CI/CD | AI-assisted DevOps

GitHub: https://github.com/anandtech-devops

LinkedIn: https://linkedin.com/in/anand-srivastava-79b51918
