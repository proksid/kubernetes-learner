# ☸️ Kubernetes CKA / CKAD Notes

![Kubernetes](https://img.shields.io/badge/Kubernetes-CKA%20%2F%20CKAD%20Notes-326CE5?logo=kubernetes&logoColor=white)
![CKA](https://img.shields.io/badge/Certification-CKA-1f6feb?logo=cncf&logoColor=white)
![CKAD](https://img.shields.io/badge/Certification-CKAD-1f6feb?logo=cncf&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Markdown](https://img.shields.io/badge/Format-Markdown-000000?logo=markdown)

> Personal **Kubernetes** study notes covering both the **CKA** (Certified Kubernetes Administrator) and **CKAD** (Certified Kubernetes Application Developer) exams - architecture, cluster installation/upgrade/backup, workloads, networking, storage, security, autoscaling, troubleshooting, and day-to-day `kubectl` workflows.

---

## 📑 Table of Contents

- [Start Here](#-start-here)
- [Contents](#-contents)
- [CKA / CKAD Exam Tips & Hints](#-cka--ckad-exam-tips--hints)
- [Coverage](#-coverage)
- [CKA / CKAD Study Resources](#-cka--ckad-study-resources)

---

## 🚀 Start Here

| Resource | Description |
|---|---|
| [`Kubernetes.md`](Kubernetes.md) | Main notes - start here |
| [`attachment/`](attachment/) | Diagrams and screenshots |

**Fastest route:** search for `CKA / CKAD hints` inside [`Kubernetes.md`](Kubernetes.md).

---

## 📂 Contents

- **[Tools](Kubernetes.md#playground-tools)** - `kubectl`, `kustomize`, `kind`, `minikube`, `kubeadm`, `Cluster API`, `kops`, `kubespray`, `MicroK8s`, `K3S`
- **[Main Components](Kubernetes.md#main-components)** - [Control Plane](Kubernetes.md#control-plane-nodes) · [Working Nodes](Kubernetes.md#working-nodes) · [Network Communication](Kubernetes.md#5-network-communication) · [Addons](Kubernetes.md#4-addons) · [Component Communication Examples](Kubernetes.md#component-communication-examples)
- **[Cluster Administration](Kubernetes.md#maintenance)** *(CKA)*
  - [Cluster Installation (kubeadm)](Kubernetes.md#1-cluster-installation) · [Cluster Upgrade](Kubernetes.md#2-cluster-upgrade) · [ETCD Backup & Restore](Kubernetes.md#3-etcd-backup)
- **[KubeConfig](Kubernetes.md#0-kubeconfig)**
- **[Resources & Workloads](Kubernetes.md#workload-resources)**
  - [Namespace](Kubernetes.md#1-namespace) · [Pods / Containers](Kubernetes.md#2-pods--containers) · [ReplicaSet](Kubernetes.md#3-replicaset-replicationcontroller)
  - [Deployment](Kubernetes.md#4-deployment) · [StatefulSet](Kubernetes.md#5-statefulset) · [DaemonSet](Kubernetes.md#6-daemonset)
  - [Job](Kubernetes.md#7-job) · [CronJob](Kubernetes.md#8-crontabjob)
- **[Platform Topics](Kubernetes.md#workload-resources)**
  - [Networking](Kubernetes.md#9-networking) - Service, Ingress, Gateway API, NetworkPolicy, EndpointSlices, Network Tools
  - [ConfigMap](Kubernetes.md#10-configmap) · [Secret](Kubernetes.md#11-secret) · [ServiceAccount](Kubernetes.md#12-serviceaccount)
  - [AAA Security](Kubernetes.md#13-aaa-security) - Authentication, Authorization, Admission Control
  - [Monitoring](Kubernetes.md#14-monitoring)
  - [Volumes](Kubernetes.md#15-volumes) - PV/PVC lifecycle, StorageClass, CSI
  - [Extensions](Kubernetes.md#16-kubernetes-extensions) · [CRDs](Kubernetes.md#17-custom-resource-definition-crd-cr-operator) · [Helm](Kubernetes.md#18-helm) · [Kustomize](Kubernetes.md#19-kustomize) · [Service Mesh](Kubernetes.md#20-service-mesh)
  - [Scaling](Kubernetes.md#21-scaling) - HPA, VPA

---

## 🛠 CKA / CKAD Exam Tips & Hints

In [`Kubernetes.md`](Kubernetes.md), **CKA / CKAD hints** are short callout blocks placed next to exam-heavy topics 
and serve as a brief memorization for quicker access to the required resources under exam pressure.

Each hint typically provides:

- A link to the relevant **Kubernetes docs** page
- **`kubectl <subcommand> -h`** - fastest lookup under time pressure
- **`kubectl explain <resource>`** - field-level detail
- **Search patterns** - keywords to find fields in manifests or docs

### Exam Speed Flow

```
CKA / CKAD hints  →  kubectl -h  →  kubectl explain  →  kubernetes.io docs
```

> ⚠️ Some resources (e.g. PV, PVC) still require a manifest template from the official docs - they cannot be generated imperatively.

---

## 📋 Coverage

- [x] Kubernetes architecture and component communication
- [x] Cluster installation (kubeadm), upgrade, and ETCD backup/restore *(CKA)*
- [x] Workloads and rollout strategies
- [x] Networking - Service, Ingress, Gateway API, NetworkPolicy, EndpointSlices
- [x] ConfigMap, Secret, ServiceAccount, RBAC, Admission controls
- [x] Volumes, PV/PVC lifecycle, StorageClass, CSI
- [x] Extensions - CRDs, Helm, Kustomize, Service Mesh
- [x] Autoscaling - HPA and VPA

---

## 📚 CKA / CKAD Study Resources

1. **[Kubernetes Docs](https://kubernetes.io/docs/home/)** *(Primary source of truth)*
   - Navigate [Concepts](https://kubernetes.io/docs/concepts/) and [Tasks](https://kubernetes.io/docs/tasks/) quickly.

2. **Udemy**
   - **[CKA](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)** · **[CKAD](https://www.udemy.com/course/certified-kubernetes-application-developer)**
   with Practice Tests by [Mumshad Mannambeth](https://www.udemy.com/user/mumshad-mannambeth/), [KodeKloud](https://www.udemy.com/user/kodekloud/), [Vijin Palazhi](https://www.udemy.com/user/vijin-palazhi-2/)
   - Structured video lessons + hands-on labs, lightning labs, and mock exams via [KodeKloud](https://kodekloud.com/).
   - [CKA Notes on GitHub](https://github.com/kodekloudhub/certified-kubernetes-administrator-course)

3. **KodeKloud - Ultimate Mock Exam Series**
   - **[CKA](https://learn.kodekloud.com/user/courses/ultimate-certified-kubernetes-administrator-cka-mock-exam-series)** · **[CKAD](https://learn.kodekloud.com/courses/ultimate-certified-kubernetes-application-developer-ckad-mock-exam-series)**
   - High-quality mock labs.

4. **KillerCoda Scenarios**
   - **[CKA](https://killercoda.com/cka)** · **[CKAD](https://killercoda.com/ckad)**
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

