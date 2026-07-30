# kube-scheduler parameters reference

> **CKA / CKAD / CKS**

## Network / serving

| Parameter        | Usual value                      | Purpose                                                                                              |
| ---------------- | -------------------------------- | ---------------------------------------------------------------------------------------------------- |
| *--bind-address* | `127.0.0.1`                      | Interface the scheduler's HTTPS metrics/healthz server listens on                                    |
| *--secure-port*  | `10259`                          | HTTPS port for `/metrics`, `/healthz`, `/readyz`; replaces the removed *--port* (HTTP, gone in 1.23) |
| *--kubeconfig*   | `/etc/kubernetes/scheduler.conf` | kubeconfig used to authenticate to the API server; absent = in-cluster ServiceAccount used instead   |
| *--master*       | *(flag value only)*              | Override API server URL; takes precedence over *--kubeconfig*; rarely needed in kubeadm clusters     |

---

## TLS / serving cert

| Parameter                | Usual value                         | Purpose                                                                   |
| ------------------------ | ----------------------------------- | ------------------------------------------------------------------------- |
| *--tls-cert-file*        | `/etc/kubernetes/pki/scheduler.crt` | Serving cert for the scheduler's own HTTPS endpoint                       |
| *--tls-private-key-file* | `/etc/kubernetes/pki/scheduler.key` | Private key matching the serving cert                                     |
| *--tls-min-version*      | flag value only                     | Minimum TLS version; set `VersionTLS12` or `VersionTLS13` for hardening   |
| *--tls-cipher-suites*    | flag value only                     | Allowed TLS 1.2 cipher suites; TLS 1.3 suites are fixed and unaffected    |
| *--client-ca-file*       | `/etc/kubernetes/pki/ca.crt`        | CA for verifying inbound client certs to the scheduler's own HTTPS server |

---

## Scheduling profile / plugin configuration

| Parameter                           | Usual value                             | Purpose                                                                                                                                                                    |
| ----------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *--config*                          | `/etc/kubernetes/scheduler-config.yaml` | Path to a `KubeSchedulerConfiguration` object (v1, stable since 1.25); **the primary way to configure the scheduler in modern clusters**; supersedes most individual flags |
| *--leader-elect*                    | `true`                                  | Enable leader election; required in HA control planes so only one scheduler instance is active                                                                             |
| *--leader-elect-lease-duration*     | `15s`                                   | How long a non-leader waits before attempting to acquire the lease                                                                                                         |
| *--leader-elect-renew-deadline*     | `10s`                                   | How long the acting leader tries to renew the lease before giving up                                                                                                       |
| *--leader-elect-retry-period*       | `2s`                                    | Interval between lease acquisition attempts by a non-leader                                                                                                                |
| *--leader-elect-resource-namespace* | `kube-system`                           | Namespace of the `Lease` object used for leader election                                                                                                                   |

> **Exam note**: In kubeadm clusters the scheduler is a static pod. Leader election flags and *--config* appear directly in *spec.containers[].command* in `/etc/kubernetes/manifests/kube-scheduler.yaml`.

---

## Scheduling algorithm tuning

| Parameter                                  | Usual value | Purpose                                                                                                                                                  |
| ------------------------------------------ | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *--percentage-of-nodes-to-score*           | `0` (auto)  | Cap on the fraction of nodes evaluated per scheduling cycle; `0` = use built-in heuristic (~5 % of nodes, min 100); raising to `100` disables early exit |
| *--pod-max-in-unschedulable-pods-duration* | `5m`        | Max time a pod stays in the unschedulable queue before being retried; increased from `1m` in 1.28                                                        |

> **Exam note**: Fine-grained plugin weights and `QueueSort`/`Filter`/`Score`/`Bind` plugin enable/disable are expressed in `KubeSchedulerConfiguration.profiles[].plugins`, not as CLI flags.

---

## Observability / hardening

| Parameter                | Usual value       | Purpose                                                                     |
| ------------------------ | ----------------- | --------------------------------------------------------------------------- |
| *--profiling*            | `true` (default)  | Exposes Go pprof at `/debug/pprof/`; set `false` in hardened clusters (CKS) |
| *--contention-profiling* | `false` (default) | Enables mutex-contention profiling; only meaningful when `--profiling=true` |
| *--v*                    | `2` (typical)     | Verbosity level for klog; higher values expose scheduling decisions in logs |

---

## Deprecated / removed flags (exam traps)

| Parameter                      | Status              | Notes                                                                                       |
| ------------------------------ | ------------------- | ------------------------------------------------------------------------------------------- |
| *--port / --address*         | **Removed in 1.23** | HTTP (insecure) endpoint; absence is correct and expected                                   |
| *--policy-config-file*       | **Removed in 1.25** | Old policy-based scheduler config; replaced entirely by `KubeSchedulerConfiguration`        |
| *--policy-configmap*         | **Removed in 1.25** | ConfigMap variant of the old policy config; same replacement                                |
| *--use-legacy-policy-config* | **Removed in 1.25** | Toggle for old policy format; no equivalent in new config                                   |
| *--algorithm-provider*       | **Removed in 1.24** | Named algorithm sets (`DefaultProvider`, `ClusterAutoscalerProvider`); replaced by profiles |

---

## `KubeSchedulerConfiguration` quick-reference

The *--config* file is the **only supported way** to configure plugins, profiles, and per-profile plugin weights in k8s ≥ 1.25.

```yaml
# /etc/kubernetes/scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1   # stable since 1.25; v1beta3 removed in 1.29
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /etc/kubernetes/scheduler.conf
leaderElection:
  leaderElect: true
profiles:
  - schedulerName: default-scheduler           # pods without schedulerName use this profile
    plugins:
      score:
        disabled:
          - name: NodeResourcesBalancedAllocation  # example: disable a default Score plugin
        enabled:
          - name: NodeResourcesFit
            weight: 2                              # boost weight of a default plugin
    pluginConfig:
      - name: NodeResourcesFit
        args:
          scoringStrategy:
            type: LeastAllocated                   # or MostAllocated, RequestedToCapacityRatio
  - schedulerName: high-priority-scheduler     # second profile; pods opt in via schedulerName
    plugins:
      filter:
        disabled:
          - name: "*"                              # disable all filters → schedule anywhere
```

Key fields for exam scenarios:

| Field                                                   | Purpose                                                                               |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| *profiles[].schedulerName*                              | Maps to pod's `spec.schedulerName`; default profile must be named `default-scheduler` |
| *profiles[].plugins.<extension-point>.enabled/disabled* | Enable, disable, or reorder plugins at a specific extension point                     |
| *profiles[].plugins.<extension-point>[].weight*         | Integer weight multiplier for Score plugins (higher = more influence)                 |
| *profiles[].pluginConfig[].args*                        | Per-plugin typed config; each plugin defines its own args struct                      |
| *leaderElection.leaderElect*                            | Mirror of *--leader-elect*; config file takes precedence over CLI flag                |

---

## Scheduling Framework extension points

Understanding which plugin type controls which phase is tested in CKA/CKAD troubleshooting scenarios.

| Extension point | Phase | What it does | Default plugins (examples) |
|---|---|---|---|
| `QueueSort` | Pre-scheduling | Orders pods in the scheduling queue | `PrioritySort` |
| `PreFilter` | Filter prep | Computes/validates pod metadata before filtering | `NodeResourcesFit`, `PodTopologySpread` |
| `Filter` | Feasibility | Eliminates nodes that cannot run the pod | `NodeUnschedulable`, `TaintToleration`, `NodeAffinity`, `VolumeBinding` |
| `PostFilter` | After failed filter | Runs when no node passes Filter (preemption lives here) | `DefaultPreemption` |
| `PreScore` | Score prep | Shares state across Score plugins | `PodTopologySpread`, `TaintToleration` |
| `Score` | Ranking | Assigns scores to feasible nodes | `NodeResourcesFit`, `ImageLocality`, `InterPodAffinity` |
| `Reserve` | Resource reservation | Marks resources as reserved before binding | `VolumeBinding` |
| `Permit` | Binding gate | Approves, denies, or waits before binding (gang scheduling) | *(none by default)* |
| `PreBind` | Pre-binding | Side effects before the actual bind (e.g. provision volumes) | `VolumeBinding` |
| `Bind` | Binding | Creates the `Binding` object to assign pod → node | `DefaultBinder` |
| `PostBind` | Post-binding | Cleanup/notification after successful bind | *(none by default)* |

---

## CKA hardening checklist

- [ ] *--profiling=false* (or set in `KubeSchedulerConfiguration`)
- [ ] *--leader-elect=true* in HA clusters (≥ 2 control-plane nodes)
- [ ] *--config* points to a valid `KubeSchedulerConfiguration` v1 file
- [ ] No *--policy-config-file* or *--algorithm-provider* (removed; presence = misconfiguration)
- [ ] Static pod manifest has correct *volumeMounts* if *--config* file is on the host filesystem
- [ ] *--tls-min-version=VersionTLS12* on the scheduler's own HTTPS endpoint
