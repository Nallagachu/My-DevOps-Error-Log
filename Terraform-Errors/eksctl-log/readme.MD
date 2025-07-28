
# 🚀 RoboShop Kubernetes Deployment: Troubleshooting & Notes

Welcome to the Kubernetes-based deployment log for **RoboShop**, a microservices-based e-commerce application. This README documents setup steps, common issues, and resolutions while deploying the app on a self-managed **EKS cluster** hosted on an **EC2 instance** using `kubectl` and Docker.

---

## 📦 Project Overview

| Key            | Details                                                        |
| -------------- | -------------------------------------------------------------- |
| **Repository** | [`k8-roboshop`](https://github.com/Nallagachu/k8-roboshop.git) |
| **Platform**   | Kubernetes on AWS EKS                                          |
| **Hosted On**  | EC2 (Terraform-provisioned)                                    |
| **Purpose**    | Deploy RoboShop microservices using individual manifest files  |
| **Tools Used** | Terraform, kubectl, Docker                                     |

---

## ✅ Initial Setup Completed

* ✅ EKS cluster launched successfully
* ✅ `kubectl` configured and able to communicate with cluster (`kubectl get nodes`)
* ✅ Docker installed and running on EC2
* ✅ EC2 instance public IP accessible
* ✅ GitHub repo connected for YAML deployment files

---

## 🧩 Step 1: Namespace Issue

### 🔍 Problem

When applying resources (e.g., `mongodb.yaml`), the following error occurred:

```bash
Error from server (NotFound): error when creating "mongodb.yaml": namespaces "roboshop" not found
```

### 🛠️ Root Cause

Namespace `roboshop` was not created before applying the resources.

### ✅ Solution

1. Created a namespace manifest file:

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
```

2. Applied it using:

```bash
kubectl apply -f namespace.yaml
```

3. Verified it with:

```bash
kubectl get ns
```

> ✅ **Best Practice**: Add `namespace: roboshop` to every manifest OR always apply using `kubectl -n roboshop`.

---

## 🔄 Next Deployment Steps (Live Checklist)

* [x] Namespace creation and validation
* [ ] Add `namespace: roboshop` in all manifest files
* [ ] Create loop/script to apply manifests in the correct order
* [ ] Skip applying `frontend` for now (until all backends are stable)
* [ ] Debug service connectivity or image issues as they arise
* [ ] Log and document each new issue and fix

---

## 🐛 Troubleshooting Log

| Error                   | Root Cause          | Fix                   |
| ----------------------- | ------------------- | --------------------- |
| *(Add next error here)* | *(Identify reason)* | *(Describe solution)* |

> This section will grow as more resources are deployed.

---

## 📘 To-Do List

* [ ] ✅ Create a Bash script to apply all manifests in proper order
* [ ] 📄 Document every fix and resolution in this README
* [ ] 🛠 Add health checks and readiness probes to manifests
* [ ] 🌐 Publish working frontend URL once available

---

## 📂 Related Repositories

* 🔗 **Docker Build Project**: [`docker`](https://github.com/Nallagachu/docker.git)
* 🔗 **This Repo**: [`k8-roboshop`](https://github.com/Nallagachu/k8-roboshop.git)

---

## 🙌 Contributing / Learning Purpose

This project is part of my continuous learning journey in **DevOps, Kubernetes, and automation**. The goal is to not only build and troubleshoot real-world infrastructure but also share the process transparently for others learning the same path.

