# 🛠️ My DevOps Error Log — Real Incidents, Clear Fixes

A curated collection of real-world DevOps incidents, root cause investigations, and proven fixes. Perfect for learning, collaboration, and demonstrating real-world skills to hiring teams.

---

## 📂 Repository Contents

Explore Markdown files like:

```
/My-DevOps-Error-Log/
├── mongodb-ebs-csi-debug.md
├── ui-service-failed-start.md
├── helm-chart-timeouts.md
└── ...
```

Each file includes:
- 🍀 Context (project, tools, environment)
- 🐞 Problem overview and symptoms
- 🔍 Diagnosis steps (logs, commands, events)
- 🛠️ Actions taken and the outcome
- 🌱 Key takeaways for future troubleshooting

---

## 🔍 Highlighted Case: MongoDB PVC Pending on AWS

**Problem**  
MongoDB pod stuck in `Pending`, PVC unbound even though values.yaml and StorageClass were set.

**Analysis**  
- `kubectl describe pvc`: stuck in Pending  
- StorageClass pointed to `ebs.csi.aws.com`  
- Missing AWS EBS CSI Driver in the cluster

**Solution**  
Installed CSI driver with:

```bash
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.30"
```

✅ PVC bound, pod launched, EBS volume provisioned successfully.

---

## ✔️ Why This Repo Matters

- Documents **real-life challenges** in production-like DevOps environments  
- Demonstrates **debug-and-fix workflows** with logs and commands  
- Serves as a learning resource for engineers and a work sample for hiring teams

---

## 🚀 Get Started

```bash
git clone https://github.com/Nallagachu/My-DevOps-Error-Log.git
cd My-DevOps-Error-Log
```

Browse through each case to follow the full diagnostic and fix narrative.

---

## 🧠 Real DevOps Lessons

| Incident | Impact |
|---------|--------|
| PVC stuck pending | Understand the role of CSI and storage classes |
| Helm install fails | Investigate chart values and resource definitions |
| ImagePullBackOff | Check secrets, network policies, and registries |
| CI build error | Review pipeline logs and artifact behavior |

---

## 🔗 Related Work

For accompanying DevOps deployments with Helm charts → [K8-RoboShop-Helm](https://github.com/Nallagachu/k8-roboshop-helm)

---

## 💬 Feedback & Collaboration

Found this helpful or want to suggest improvements?  
Ideas or feedback are welcome—I'm all ears!

---

> Real DevOps isn’t theoretical — it’s fixing issues that don’t show up in docs.