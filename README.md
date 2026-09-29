# K8sGPT Kubernetes AI Troubleshooting

AI-assisted Kubernetes troubleshooting using **K8sGPT, Ollama, and Llama 3.2** on an AWS EC2-based Kubernetes cluster.

---

## 📌 Project Overview

This project demonstrates how AI can assist a DevOps Engineer in troubleshooting Kubernetes issues.

An intentional Kubernetes image-pull failure was created using an invalid Docker image tag. **K8sGPT** was used to analyze the failed Kubernetes Pod, while **Ollama running Llama 3.2** provided a human-readable explanation of the issue.

The AI-generated recommendation was manually validated before applying the Kubernetes fix.

---

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

### How It Works

1. A Kubernetes Deployment is created with an invalid Docker image.
2. The Pod enters `ImagePullBackOff`.
3. `kubectl` is used for initial troubleshooting.
4. K8sGPT analyzes the failed Pod.
5. K8sGPT sends the analysis to Ollama.
6. Ollama runs the Llama 3.2 model locally.
7. The AI provides a human-readable explanation.
8. The recommendation is manually validated.
9. The Kubernetes image is corrected.
10. The Pod is verified in the `Running` state.

---

## 🛠️ Technologies Used

* AWS EC2
* Kubernetes
* kubeadm
* kubelet
* kubectl
* containerd
* Flannel CNI
* K8sGPT
* Ollama
* Llama 3.2
* Docker / Docker Hub
* Linux
* Windows PowerShell
* OpenSSH
* SSH Reverse Tunnel

---

## ☁️ Infrastructure

### AWS EC2

* Platform: AWS EC2
* Operating System: Ubuntu
* Kubernetes: v1.34.x
* Container Runtime: containerd
* Network Plugin: Flannel
* Cluster Type: Single-node Kubernetes cluster

### Local AI Environment

* Operating System: Windows
* Ollama
* Model: `llama3.2:latest`
* Model Parameters: 3.2B
* Quantization: Q4_K_M

The LLM was kept on the local Windows machine instead of the EC2 instance to avoid running a resource-intensive AI model on the Kubernetes server.

---

## 🚀 Kubernetes Setup

A single-node Kubernetes cluster was created using `kubeadm`.

The cluster was configured with:

* containerd
* kubeadm
* kubelet
* kubectl
* Flannel CNI

### Verify Cluster

```bash
kubectl get nodes
```

Example:

```text
NAME                   STATUS   ROLES           VERSION
ip-172-31-38-224       Ready    control-plane   v1.34.x
```

---

## 🤖 K8sGPT Installation

K8sGPT was installed on the Kubernetes EC2 instance.

### Check Version

```bash
k8sgpt version
```

Example:

```text
k8sgpt: 0.4.36
```

### Check AI Providers

```bash
k8sgpt auth list
```

Ollama was configured as the active K8sGPT provider.

---

## 🔗 Ollama Integration

Ollama was running locally on the Windows machine:

```text
127.0.0.1:11434
```

The Llama model used in this project:

```text
llama3.2:latest
```

### Configure Ollama in K8sGPT

```bash
k8sgpt auth add --backend ollama \
  --model llama3.2 \
  --baseurl http://127.0.0.1:11434
```

Set Ollama as the default provider:

```bash
k8sgpt auth default --provider ollama
```

### Verify Configuration

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

---

## 🔐 Secure SSH Reverse Tunnel

The Ollama API was **not exposed directly to the internet**.

Instead, an encrypted SSH reverse tunnel was created from the Windows machine to the AWS EC2 instance.

```powershell
ssh -i "C:\path\to\your-key" `
  -N `
  -R 11434:127.0.0.1:11434 `
  ubuntu@<EC2_PUBLIC_IP>
```

This allowed the Kubernetes EC2 instance to access the local Ollama service through:

```text
127.0.0.1:11434
```

### Verify Ollama Connectivity from EC2

```bash
curl http://127.0.0.1:11434/api/tags
```

The response confirmed that the `llama3.2:latest` model was available.

### Security Benefit

Port `11434` did not need to be publicly exposed through the AWS Security Group.

---

## 🧪 Creating an Intentional Kubernetes Failure

A test namespace was created:

```bash
kubectl create namespace k8sgpt-lab
```

An intentionally broken Deployment was created using a non-existent Docker image tag:

```bash
kubectl create deployment broken-app \
  --image=nginx:this-image-does-not-exist \
  -n k8sgpt-lab
```

### Check Pod Status

```bash
kubectl get pods -n k8sgpt-lab
```

Expected result:

```text
NAME                          READY   STATUS
broken-app-xxxxxxxxxx-xxxxx  0/1     ImagePullBackOff
```

The Pod initially entered:

```text
ErrImagePull
ImagePullBackOff
```

---

## 🔍 Manual Troubleshooting

The failed Pod was investigated using:

```bash
kubectl describe pod <pod-name> -n k8sgpt-lab
```

The Kubernetes events showed an image-pull failure:

```text
Failed to pull image
nginx:this-image-does-not-exist
```

The image tag did not exist, so Kubernetes repeatedly attempted to pull the image and eventually entered `ImagePullBackOff`.

---

## 🧠 K8sGPT AI-Assisted Troubleshooting

K8sGPT was used to analyze the failing Pod:

```bash
k8sgpt analyze \
  --filter=Pod \
  --namespace=k8sgpt-lab \
  --explain
```

K8sGPT identified the problem:

```text
Back-off pulling image "nginx:this-image-does-not-exist"

ErrImagePull

failed to resolve image:
docker.io/library/nginx:this-image-does-not-exist
```

The Ollama LLM generated a human-readable explanation and suggested checking the Docker image and updating the Kubernetes Deployment.

### Important Approach

The AI recommendation was **not blindly applied**.

The Kubernetes error was manually validated using:

```bash
kubectl describe pod
```

This demonstrates the use of AI as a **troubleshooting assistant**, while the DevOps Engineer remains responsible for validating and applying the fix.

---

## 🔧 Fix

Before updating the Deployment, the actual Kubernetes container name was verified:

```bash
kubectl get deployment broken-app \
  -n k8sgpt-lab \
  -o jsonpath='{.spec.template.spec.containers[*].name}{"\n"}'
```

The container name was:

```text
nginx
```

The invalid image was then replaced with a valid image:

```bash
kubectl set image deployment/broken-app \
  nginx=nginx:latest \
  -n k8sgpt-lab
```

---

## ✅ Verification

The Deployment was verified after applying the fix:

```bash
kubectl get pods -n k8sgpt-lab
```

Final result:

```text
NAME                          READY   STATUS    RESTARTS
broken-app-xxxxxxxxxx-xxxxx  1/1     Running   0
```

The Kubernetes Deployment successfully recovered from:

```text
ImagePullBackOff
```

to:

```text
Running
```

---

## 🔄 Troubleshooting Workflow

```text
              Kubernetes Failure
                     │
                     ▼
              kubectl get pods
                     │
                     ▼
              ImagePullBackOff
                     │
                     ▼
            kubectl describe pod
                     │
                     ▼
               K8sGPT analyze
                     │
                     ▼
          AI explanation using Ollama
                     │
                     ▼
       Engineer validates recommendation
                     │
                     ▼
          Correct Kubernetes image
                     │
                     ▼
             Rollout verification
                     │
                     ▼
                Pod Running
```

---

## 🎯 Key Learnings

* Kubernetes Pod troubleshooting
* `ErrImagePull` and `ImagePullBackOff`
* Kubernetes Deployments
* Kubernetes container configuration
* K8sGPT AI-assisted troubleshooting
* Ollama local LLM integration
* Llama 3.2
* SSH reverse tunneling
* AWS EC2-based Kubernetes
* Manual validation of AI-generated recommendations
* Using AI as a troubleshooting assistant rather than blindly applying AI-generated changes

---

## 📸 Project Screenshots

Recommended screenshots for this repository:

### 1. Kubernetes Failure

Show:

```bash
kubectl get pods -n k8sgpt-lab
```

with:

```text
ImagePullBackOff
```

### 2. K8sGPT AI Analysis

Show:

```bash
k8sgpt analyze --filter=Pod --namespace=k8sgpt-lab --explain
```

with:

```text
AI Provider: ollama
```

### 3. Ollama Model

Show:

```bash
ollama list
```

with:

```text
llama3.2:latest
```

### 4. Successful Recovery

Show:

```bash
kubectl get pods -n k8sgpt-lab
```

with:

```text
1/1   Running
```

---

## ⚠️ Security Notes

* No private keys are included in this repository.
* No AWS credentials are included.
* No passwords or secrets are included.
* The actual EC2 public IP is not stored in the documentation.
* The Ollama API was not exposed publicly.
* A secure SSH reverse tunnel was used for communication between AWS EC2 and the local Ollama service.
* Sensitive configuration values should always be stored securely and never committed to Git.

---

## 👨‍💻 Author

**Anand Srivastava**

**DevOps Engineer**

AWS | Kubernetes | Docker | Jenkins | CI/CD | AI-assisted DevOps

GitHub: https://github.com/anandtech-devops

LinkedIn: https://linkedin.com/in/anand-srivastava-79b51918

