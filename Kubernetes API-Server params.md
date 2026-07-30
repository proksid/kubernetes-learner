# kube-apiserver parameters reference

> **CKA / CKAD / CKS**

## Network

| Parameter                    | Usual location                   | Purpose                                                                                        |
| ---------------------------- | -------------------------------- | ---------------------------------------------------------------------------------------------- |
| *--advertise-address*        | node IP (control-plane NIC)      | IP the API server advertises to the rest of the cluster; what other components use to reach it |
| *--bind-address*             | `0.0.0.0`                        | Network interface the API server listens on; pin to a specific IP on multi-homed nodes         |
| *--secure-port*              | `6443`                           | HTTPS port; *--insecure-port* was removed in 1.20                                              |
| *--service-cluster-ip-range* | `10.96.0.0/12` (kubeadm default) | CIDR pool for ClusterIP Service allocation; must not overlap pod or node CIDRs                 |
| *--service-node-port-range*  | `30000-32767`                    | Port range for NodePort Services                                                               |
| *--cors-allowed-origins*     | flag value only                  | Regex list of origins allowed for CORS; relevant for browser-based API access                  |

---

## TLS / serving cert

| Parameter                | Usual location                      | Purpose                                                                          |
| ------------------------ | ----------------------------------- | -------------------------------------------------------------------------------- |
| *--tls-cert-file*        | `/etc/kubernetes/pki/apiserver.crt` | API server's own serving certificate presented to clients                        |
| *--tls-private-key-file* | `/etc/kubernetes/pki/apiserver.key` | Private key matching the serving cert                                            |
| *--tls-min-version*      | flag value only                     | Minimum TLS version accepted; set `VersionTLS12` or `VersionTLS13` for hardening |
| *--tls-cipher-suites*    | flag value only                     | Allowed TLS 1.2 cipher suites; TLS 1.3 suites are fixed and unaffected           |

---

## Authentication

| Parameter                            | Usual location                           | Purpose                                                                                                   |
| ------------------------------------ | ---------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| *--client-ca-file*                   | `/etc/kubernetes/pki/ca.crt`             | CA bundle for verifying inbound client X.509 certs; CN→username, O→groups                                 |
| *--requestheader-client-ca-file*     | `/etc/kubernetes/pki/front-proxy-ca.crt` | CA for verifying aggregation-layer front-proxy certs only; separate trust root from `--client-ca-file`    |
| *--requestheader-allowed-names*      | flag value only                          | CN values allowed in front-proxy certs; empty = any CN accepted                                           |
| *--requestheader-username-headers*   | `X-Remote-User`                          | Header the front-proxy injects to carry the end-user's username                                           |
| *--requestheader-group-headers*      | `X-Remote-Group`                         | Header the front-proxy injects to carry the end-user's groups                                             |
| *--service-account-key-file*         | `/etc/kubernetes/pki/sa.pub`             | Public key(s) used to verify ServiceAccount JWT signatures                                                |
| *--service-account-signing-key-file* | `/etc/kubernetes/pki/sa.key`             | Private key the API server uses to sign ServiceAccount tokens (required since 1.20)                       |
| *--service-account-issuer*           | `https://kubernetes.default.svc`         | `iss` claim embedded in ServiceAccount JWTs; must match external OIDC verifier expectations               |
| *--oidc-issuer-url*                  | external IdP URL                         | OIDC provider for human user authn; API server fetches JWKS from `<url>/.well-known/openid-configuration` |
| *--oidc-client-id*                   | flag value only                          | Expected `aud` claim in OIDC tokens; required alongside *--oidc-issuer-url*                               |
| *--oidc-username-claim*              | `sub` (default)                          | JWT claim mapped to Kubernetes username; commonly set to `email`                                          |
| *--oidc-groups-claim*                | flag value only                          | JWT claim mapped to Kubernetes groups                                                                     |
| *--anonymous-auth*                   | `true` (default)                         | Allows unauthenticated requests as `system:anonymous`; disabling can break health probes                  |
| *--token-auth-file*                  | *(absent in secure clusters)*            | Static CSV token file; **removed in 1.34**; presence is a CKS red flag                                    |

---

## Authorization

| Parameter                             | Usual location                       | Purpose                                                                                                            |
| ------------------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| *--authorization-mode*                | `Node,RBAC`                          | Ordered list of authorizers; Node must precede RBAC so kubelet Node authorizer fires first                         |
| *--authorization-webhook-config-file* | `/etc/kubernetes/authz-webhook.yaml` | kubeconfig for external webhook authorizer (OPA, custom engines); used when `Webhook` is in *--authorization-mode* |

---

## Admission control

| Parameter                         | Usual location                          | Purpose                                                                                            |
| --------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------- |
| *--enable-admission-plugins*      | flag value only                         | Plugins added on top of defaults; key ones: `NodeRestriction`, `PodSecurity`, `AlwaysPullImages`   |
| *--disable-admission-plugins*     | flag value only                         | Plugins to remove; disabling `NodeRestriction` is a CKS misconfiguration red flag                  |
| *--admission-control-config-file* | `/etc/kubernetes/admission-config.yaml` | Per-plugin config (e.g. PodSecurity cluster-wide defaults); uses `AdmissionConfiguration` API kind |

---

## etcd connectivity

| Parameter                      | Usual location                                  | Purpose                                                                              |
| ------------------------------ | ----------------------------------------------- | ------------------------------------------------------------------------------------ |
| *--etcd-servers*               | `https://127.0.0.1:2379`                        | etcd endpoint(s); multiple entries for HA; stacked topology uses localhost           |
| *--etcd-certfile*              | `/etc/kubernetes/pki/apiserver-etcd-client.crt` | Client cert the API server presents to etcd for mTLS                                 |
| *--etcd-keyfile*               | `/etc/kubernetes/pki/apiserver-etcd-client.key` | Private key matching the etcd client cert                                            |
| *--etcd-cafile*                | `/etc/kubernetes/pki/etcd/ca.crt`               | CA the API server uses to verify etcd's server cert; separate from cluster CA        |
| *--encryption-provider-config* | `/etc/kubernetes/encryption-config.yaml`        | Enables at-rest encryption of etcd data; absent = Secrets stored as base64 plaintext |

---

## Audit logging

| Parameter                     | Usual location                       | Purpose                                                                                     |
| ----------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------- |
| *--audit-policy-file*         | `/etc/kubernetes/audit-policy.yaml`  | Audit `Policy` object; defines what events to log and at which level                        |
| *--audit-log-path*            | `/var/log/kubernetes/audit.log`      | File sink for audit events; required alongside *--audit-policy-file* to actually write logs |
| *--audit-log-maxage*          | flag value only                      | Max days to retain rotated audit log files                                                  |
| *--audit-log-maxbackup*       | flag value only                      | Max number of retained rotated audit log files                                              |
| *--audit-log-maxsize*         | flag value only                      | Max size in MB before audit log is rotated                                                  |
| *--audit-webhook-config-file* | `/etc/kubernetes/audit-webhook.yaml` | kubeconfig for shipping audit events to an external receiver (e.g. Falco)                   |

---

## Kubelet connectivity / API client side

| Parameter                         | Usual location                                     | Purpose                                                                            |
| --------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------- |
| *--kubelet-client-certificate*    | `/etc/kubernetes/pki/apiserver-kubelet-client.crt` | Cert the API server presents when it calls kubelet APIs (logs, exec, port-forward) |
| *--kubelet-client-key*            | `/etc/kubernetes/pki/apiserver-kubelet-client.key` | Private key matching the kubelet client cert                                       |
| *--kubelet-certificate-authority* | `/etc/kubernetes/pki/ca.crt`                       | CA the API server uses to verify kubelet serving certs                             |

---

## Miscellaneous / hardening

| Parameter                          | Usual location   | Purpose                                                                                      |
| ---------------------------------- | ---------------- | -------------------------------------------------------------------------------------------- |
| *--allow-privileged*               | `false`          | Permits privileged containers; superseded by PodSecurity admission; `true` is a CKS red flag |
| *--profiling*                      | `true` (default) | Exposes Go pprof at `/debug/pprof/`; set `false` in hardened clusters                        |
| *--request-timeout*                | `1m0s`           | Global timeout for non-watch API requests                                                    |
| *--max-requests-inflight*          | `400`            | Max concurrent non-mutating requests; superseded by API Priority and Fairness in ≥ 1.29      |
| *--max-mutating-requests-inflight* | `200`            | Max concurrent mutating requests; same APF caveat                                            |
| *--feature-gates*                  | flag value only  | Enable/disable alpha or beta features by name; most gates relevant pre-1.34 are now GA       |
| *--runtime-config*                 | flag value only  | Enables or disables built-in API groups/versions; e.g. `api/all=true`                        |

---

## PKI quick-reference

Four independent trust domains under `/etc/kubernetes/pki/`:

| Trust domain | CA file | Signing keypair | Covers |
|---|---|---|---|
| Cluster | `ca.crt` | `ca.key` | Kubelet client certs, user certs, controller/scheduler certs |
| etcd | `etcd/ca.crt` | `etcd/ca.key` | etcd peer and client certs |
| Front-proxy | `front-proxy-ca.crt` | `front-proxy-ca.key` | Aggregation-layer front-proxy certs |
| Service accounts | `sa.pub` / `sa.key` | *(not a CA — raw RSA keypair)* | ServiceAccount JWT signing and verification |

## Audit log — minimum viable config

Both flags are required. Missing either produces no output even if the other is set.

```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml (static pod)
spec:
  containers:
  - command:
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
    - --audit-log-path=/var/log/kubernetes/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    volumeMounts:
    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit-policy
      readOnly: true
    - mountPath: /var/log/kubernetes/audit.log
      name: audit-log
  volumes:
  - hostPath:
      path: /etc/kubernetes/audit-policy.yaml
      type: File
    name: audit-policy
  - hostPath:
      path: /var/log/kubernetes/audit.log
      type: FileOrCreate
    name: audit-log
```

## CKS hardening checklist

- [ ] *--tls-min-version=VersionTLS12*
- [ ] *--tls-cipher-suites* set to approved list (ECDHE+AES-GCM families)
- [ ] *--anonymous-auth=false* (verify health probes still work)
- [ ] *--authorization-mode=Node,RBAC* (Node before RBAC; no AlwaysAllow)
- [ ] *--enable-admission-plugins* includes *NodeRestriction* and *PodSecurity*
- [ ] *--encryption-provider-config* present and secrets verified encrypted in etcd
- [ ] *--audit-policy-file* and *--audit-log-path* both set
- [ ] *--profiling=false*
- [ ] *--token-auth-file* absent (removed in 1.34; flag presence = fail)
- [ ] *--allow-privileged8 absent or `false`
