# ☸️ Kubernetes CKAD Notes

![Kubernetes](https://img.shields.io/badge/Kubernetes-CKAD%20Notes-326CE5?logo=kubernetes&logoColor=white)
![CKAD](https://img.shields.io/badge/Certification-CKAD-1f6feb?logo=cncf&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Markdown](https://img.shields.io/badge/Format-Markdown-000000?logo=markdown)

> Personal **Kubernetes** study notes focused on the **CKAD** (Certified Kubernetes Application Developer) exam - covering architecture, workloads, networking, storage, security, troubleshooting, and day-to-day `kubectl` workflows.

---

## 📑 Table of Contents

- [Start Here](#-start-here)
- [Contents](#-contents)
- [CKAD Exam Tips & Hints](#-ckad-exam-tips--hints)
- [Coverage](#-coverage)
- [CKAD Study Resources](#-ckad-study-resources)

---

## 🚀 Start Here

| Resource | Description |
|---|---|
| [`Kubernetes.md`](Kubernetes.md) | Main notes - start here |
| [`attachment/`](attachment/) | Diagrams and screenshots |

**Fastest route:** search for `CKAD hints` inside [`Kubernetes.md`](Kubernetes.md).

---

## 📂 Contents

- **[Tools](Kubernetes.md#tools)** - `kubectl`, `kustomize`, `kind`, `minikube`, `kubeadm`, `kops`, `kubespray`, `MicroK8s`, `K3S`
- **[Main Components](Kubernetes.md#main-components)** - [Control Plane](Kubernetes.md#control-plane-nodes) · [Working Nodes](Kubernetes.md#working-nodes) · [Network Communication](Kubernetes.md#5-network-communication) · [Addons](Kubernetes.md#4-addons)
- **[Resources & Workloads](Kubernetes.md#resources)**
  - [Namespace](Kubernetes.md#1-namespace) · [Pods / Containers](Kubernetes.md#2-pods--containers) · [ReplicaSet](Kubernetes.md#3-replicaset-replicationcontroller)
  - [Deployment](Kubernetes.md#4-deployment) · [StatefulSet](Kubernetes.md#5-statefulset) · [DaemonSet](Kubernetes.md#6-daemonset)
  - [Job](Kubernetes.md#7-job) · [CronJob](Kubernetes.md#8-crontabjob)
- **[Platform Topics](Kubernetes.md#resources)**
  - [Networking](Kubernetes.md#9-networking) - Service, Ingress, NetworkPolicy, EndpointSlices
  - [ConfigMap](Kubernetes.md#10-configmap) · [Secret](Kubernetes.md#11-secret) · [ServiceAccount](Kubernetes.md#12-serviceaccount)
  - [AAA Security](Kubernetes.md#13-aaa-security) - Authentication, Authorization, Admission Control
  - [Monitoring](Kubernetes.md#14-monitoring)
  - [Volumes](Kubernetes.md#15-volumes) - PV/PVC lifecycle, StorageClass, CSI
  - [Extensions](Kubernetes.md#16-kubernetes-extensions) · [CRDs](Kubernetes.md#17-custom-resource-definition-crd) · [Helm](Kubernetes.md#18-helm) · [Kustomize](Kubernetes.md#19-kustomize) · [Service Mesh](Kubernetes.md#20-service-mesh)

---

## 🛠 CKAD Exam Tips & Hints

In [`Kubernetes.md`](Kubernetes.md), **CKAD hints** are short callout blocks placed next to exam-heavy topics 
and serve as a brief memorization for quicker access to the required resources under exam pressure.

Each hint typically provides:

- A link to the relevant **Kubernetes docs** page
- **`kubectl <subcommand> -h`** - fastest lookup under time pressure
- **`kubectl explain <resource>`** - field-level detail
- **Search patterns** - keywords to find fields in manifests or docs

### Exam Speed Flow

```
CKAD hints  →  kubectl -h  →  kubectl explain  →  kubernetes.io docs
```

> ⚠️ Some resources (e.g. PV, PVC) still require a manifest template from the official docs - they cannot be generated imperatively.

---

## 📋 Coverage

- [x] Kubernetes architecture and component communication
- [x] Workloads and rollout strategies
- [x] Networking - Service, Ingress, NetworkPolicy, EndpointSlices
- [x] ConfigMap, Secret, ServiceAccount, RBAC, Admission controls
- [x] Volumes, PV/PVC lifecycle, StorageClass, CSI
- [x] Extensions - CRDs, Helm, Kustomize, Service Mesh

---

## 📚 CKAD Study Resources

1. **[Kubernetes Docs](https://kubernetes.io/docs/home/)** *(Primary source of truth)*
   - Navigate [Concepts](https://kubernetes.io/docs/concepts/) and [Tasks](https://kubernetes.io/docs/tasks/) quickly.

2. **[Udemy - CKAD with Tests](https://www.udemy.com/course/certified-kubernetes-application-developer)**
   by [Mumshad Mannambeth](https://www.udemy.com/user/mumshad-mannambeth/), [KodeKloud](https://www.udemy.com/user/kodekloud/), [Vijin Palazhi](https://www.udemy.com/user/vijin-palazhi-2/)
   - Structured video lessons + hands-on labs, lightning labs, and mock exams via [KodeKloud](https://kodekloud.com/).

3. **[KodeKloud - Ultimate CKAD Mock Exam Series](https://learn.kodekloud.com/courses/ultimate-certified-kubernetes-application-developer-ckad-mock-exam-series)**
   - High-quality mock labs.

4. **[KillerCoda CKAD Scenarios](https://killercoda.com/ckad)**
   - Browser-based interactive scenarios.

5. **[KillerShell](https://killer.sh/ckad)**
   - Real exam environment simulator; great for timing and muscle-memory training.

6. **[PluralSight CKAD Path](https://app.pluralsight.com/paths/certificate/certified-kubernetes-application-developer-ckad-2023)**
   - Additional learning path.

7. **Exam experience & tips:**
   - [My two cents on passing CKAD in 2022](https://kavinduchamiran.medium.com/my-two-cents-on-passing-ckad-in-2022-ffbb7f1c65be)
   - [Passing CKAD - Cheatsheet, Notes & Tips](https://medium.com/@codebob75/passing-ckad-cheatsheet-notes-and-tips-1aa285e6a473)

8. **[Ivan Velichko's iximiuz Lab](https://iximiuz.com/en/)**
   - Container internals explained with surgical precision.

