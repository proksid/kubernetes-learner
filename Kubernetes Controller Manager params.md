# kube-controller-manager parameters reference

> **CKA / CKS**

## Serving / Network

| Parameter        | Usual location        | Purpose                                                                                                      |
| ---------------- | --------------------- | ------------------------------------------------------------------------------------------------------------ |
| *--bind-address* | `127.0.0.1` (kubeadm) | Interface the HTTPS metrics/health endpoint listens on; kubeadm pins to loopback `0.0.0.0` is a CKS red flag |
| *--secure-port*  | `10257`               | HTTPS port for `/metrics`, `/healthz`, `/readyz`; the old plain-HTTP *--port=10252* was removed in 1.22      |

---

## API server connectivity / client side

| Parameter      | Usual location                            | Purpose                                                                                                        |
| -------------- | ----------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| *--kubeconfig* | `/etc/kubernetes/controller-manager.conf` | the controller-manager kubeconfig, it uses to authenticate to the API server; embeds cluster CA + client cert. |
|                |                                           |                                                                                                                |
To extract the client cert:
```bash
kubectl --kubeconfig=/etc/kubernetes/controller-manager.conf \
        config view --raw -o jsonpath='{.users[0].user.client-certificate-data}' | base64 -d
```

---

## PKI / cluster certificate signing

| Parameter                     | Usual location               | Purpose                                                                                                                                         |
| ----------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| *--cluster-signing-cert-file* | `/etc/kubernetes/pki/ca.crt` | CA cert used to issue certificates approved via the CSR API (kubelet TLS bootstrap, user certs)                                                 |
| *--cluster-signing-key-file*  | `/etc/kubernetes/pki/ca.key` | Private key for the signing CA; compromise = full cluster PKI compromise                                                                        |
| *--cluster-signing-duration*  | `8760h0m0s` (1 year)         | Validity period of certs issued through the CSR pipeline; was *--experimental-cluster-signing-duration* before 1.19, old name is gone in ≥ 1.34 |
| *--root-ca-file*              | `/etc/kubernetes/pki/ca.crt` | CA bundle injected into every ServiceAccount token Secret so in-cluster pods can verify the API server TLS cert                                 |

---

## ServiceAccount token signing

| Parameter                            | Usual location               | Purpose                                                                                                                                                                           |
| ------------------------------------ | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *--service-account-private-key-file* | `/etc/kubernetes/pki/sa.key` | Private RSA/ECDSA key used to sign ServiceAccount JWTs; the public half (`sa.pub`) is loaded by *--service-account-key-file* on the API server for verification                   |
| *--use-service-account-credentials*  | `true` (kubeadm)             | Each built-in controller gets its own dedicated ServiceAccount token instead of sharing the controller-manager's identity; required for RBAC least-privilege; **CKS audit check** |

> **Key relationship:** The controller-manager *signs* tokens (`sa.key`), the API server *verifies* them (`sa.pub`). These must be a matched keypair. Rotating `sa.key` invalidates all existing ServiceAccount tokens cluster-wide.


---

## Node lifecycle / eviction

| Parameter                        | Usual location | Purpose                                                                                                                                                         |
| -------------------------------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *--node-monitor-period*          | `5s`           | How often the node lifecycle controller polls NodeStatus via the API server                                                                                     |
| *--node-monitor-grace-period*    | `40s`          | How long a node may be unresponsive before it is marked `NotReady`; must be > `kubelet.nodeStatusUpdateFrequency × retries` or you get false-positive evictions |
| *--node-startup-grace-period*    | `1m0s`         | Extended grace for newly joining nodes before they are classified unhealthy                                                                                     |
| *--node-eviction-rate*           | `0.1`          | Nodes/second from which pods are evicted during a *healthy*-zone failure                                                                                        |
| *--secondary-node-eviction-rate* | `0.01`         | Nodes/second during an *unhealthy*-zone failure; implicitly forced to `0` when cluster size ≤ *--large-cluster-size-threshold*                                  |
| *--large-cluster-size-threshold* | `50`           | Node count above which zone-aware eviction logic kicks in                                                                                                       |
| *--unhealthy-zone-threshold*     | `0.55`         | Fraction of `NotReady` nodes in a zone that triggers "unhealthy zone" mode and drops eviction to *--secondary-node-eviction-rate*                               |

> **Note:** *--pod-eviction-timeout* was removed in 1.27. Eviction timing is now controlled by taint-based eviction tolerations on Pods combined with *--node-monitor-grace-period*.

---

## Pod CIDR / node IPAM

| Parameter                    | Usual location | Purpose                                                                                                                                                             |
| ---------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *--allocate-node-cidrs*      | `false`        | When `true`, the controller assigns a per-node pod CIDR from *--cluster-cidr*; required for in-tree IPAM; CNI plugins that manage their own IPAM leave this `false` |
| *--cluster-cidr*             | *(empty)*      | **Pod address space**; only active when *--allocate-node-cidrs=true*; must not overlap service or node CIDRs; must match the CNI appropriate parameter.             |
| *--node-cidr-mask-size*      | `24`           | Prefix length carved out of *--cluster-cidr* per node (e.g. `/24` → 254 pod IPs per node).                                                                          |
| *--service-cluster-ip-range* | `10.96.0.0/12` | Must match **kube-apiserver**'s value; used by controllers that need to know the Service CIDR                                                                       |

---

## Controller selection

| Parameter       | Usual location | Purpose                                                                                                                                        |
| --------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| *--controllers* | `*`            | Comma-separated list; `*` enables all defaults; prefix with `-` to disable (e.g. `-bootstrapsigner`); explicit names to enable a strict subset |

Controllers **disabled by default** that must be explicitly enabled: `bootstrapsigner`, `tokencleaner`.

Key named controllers for exam awareness:

| Controller name             | What it does                                                                                                          |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `csrapproving`              | Auto-approves kubelet bootstrap CSRs matching built-in RBAC rules                                                     |
| `csrsigning`                | Signs approved CSRs using *--cluster-signing-cert-file* / *--cluster-signing-key-file*                                |
| `csrcleaner`                | Garbage-collects expired CSR objects                                                                                  |
| `nodelifecycle`             | Marks nodes `NotReady`, applies `node.kubernetes.io/not-ready` and `unreachable` taints, triggers pod eviction        |
| `serviceaccount`            | Creates default ServiceAccounts in new namespaces                                                                     |
| `serviceaccount-token`      | Creates and rotates ServiceAccount token Secrets (legacy; projected tokens via TokenRequest are preferred since 1.22) |
| `garbagecollector`          | Cascading deletion of owned objects; must be in sync with API server's *--enable-garbage-collector*                   |
| `persistentvolume-binder`   | Binds PVs to PVCs (static provisioning) and triggers dynamic provisioning                                             |
| `persistentvolume-expander` | Handles PVC resize requests                                                                                           |
| `namespace`                 | Finalizes namespace deletion by removing all resources within it                                                      |
| `resourcequota`             | Enforces ResourceQuota objects; syncs usage periodically                                                              |
| `ttl-after-finished`        | Cleans up finished Jobs after their `ttlSecondsAfterFinished`                                                         |

---

## Leader election (HA control plane)

| Parameter                       | Usual location | Purpose                                                                                                                                                         |
| ------------------------------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *--leader-elect*                | `true`         | Enables leader election via a `coordination.k8s.io/v1 Lease` object; only the leader runs reconciliation loops; **must be `true` in any kubeadm or HA cluster** |
| *--leader-elect-lease-duration* | `15s`          | How long a non-leader waits after observing a stale lease before trying to acquire it                                                                           |
| *--leader-elect-renew-deadline* | `10s`          | How long the acting leader retries renewal before giving up; must be ≤ *--leader-elect-lease-duration*                                                          |
| *--leader-elect-retry-period*   | `2s`           | Interval between acquisition/renewal attempts                                                                                                                   |

## Miscellaneous / hardening

| Parameter                       | Usual location   | Purpose                                                                                                  |
| ------------------------------- | ---------------- | -------------------------------------------------------------------------------------------------------- |
| *--profiling*                   | `true` (default) | Exposes Go pprof at `/debug/pprof/`; set `false` in hardened clusters - same target as on the API server |
| *--feature-gates*               | flag value only  | Enable/disable alpha or beta controller features by name; most gates relevant before 1.34 are now GA     |
| *--concurrent-deployment-syncs* | `5`              | Reconciliation parallelism for the Deployment controller; tune up for large clusters                     |
| *--concurrent-replicaset-syncs* | `5`              | Same for ReplicaSet controller                                                                           |

---

## Static pod location (kubeadm)

```
/etc/kubernetes/manifests/kube-controller-manager.yaml
```

Inspect live flags:
```bash
# Via kubectl
kubectl -n kube-system get pod kube-controller-manager-<node> -o yaml | grep -A100 'command:'

# Via the /flagz endpoint (available since k8s 1.33)
curl -sk https://127.0.0.1:10257/flagz
```

---

## PKI quick-reference (controller-manager perspective)

| Keypair                | File                                  | Role in controller-manager                                                              |
| ---------------------- | ------------------------------------- | --------------------------------------------------------------------------------------- |
| Cluster CA             | `pki/ca.crt` + `pki/ca.key`           | *--cluster-signing-cert-file* / *--cluster-signing-key-file* - signs CSR-approved certs |
| Cluster CA (read-only) | `pki/ca.crt`                          | *--root-ca-file* - injected into ServiceAccount Secrets                                 |
| ServiceAccount keypair | `pki/sa.key` (private)                | *--service-account-private-key-file* - signs ServiceAccount JWTs                        |
| kubeconfig client cert | embedded in `controller-manager.conf` | mTLS identity when calling the API server; CN = *system:kube-controller-manager*        |

---

## CKS hardening checklist

- [ ] *--use-service-account-credentials=true*
- [ ] *--profiling=false*
- [ ] *--bind-address=127.0.0.1* (not `0.0.0.0`)
- [ ] *--leader-elect=true* (never disable in kubeadm clusters)
- [ ] *--cluster-signing-cert-file* and *--cluster-signing-key-file* present and pointing at cluster CA
- [ ] *--root-ca-file* set so in-cluster pods can verify the API server
- [ ] *--service-account-private-key-file* present and matched to *--service-account-key-file* (public half) on the API server
- [ ] No *--controllers* exclusions that remove security-critical loops (`nodelifecycle`, `garbagecollector`, `serviceaccount`)
