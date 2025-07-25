## `etcd` – Kubernetes’ Key-Value Store

---

### 🔍 What is `etcd`?

* `etcd` is a **distributed**, **consistent**, and **highly available** **key-value store**.
* It's the **source of truth** for Kubernetes — it stores all **cluster state and configuration data**.
* Think of it as the **brain** of your Kubernetes cluster.
* Every information you see when you run the kubectl get command is from the ETCD Server.

---

### 📦 What Does `etcd` Store?

| Data Type                  | Example                       |
| -------------------------- | ----------------------------- |
| Cluster state              | Node info, health, roles      |
| Pod/Deployment definitions | All YAML manifest data        |
| Secrets & ConfigMaps       | Secure and config values      |
| Resource quotas            | CPU/memory limits             |
| Role-based access control  | RBAC policies                 |
| Service discovery info     | Internal DNS and service maps |

---

### 🔐 Key Properties

| Property        | Explanation                                                            |
| --------------- | ---------------------------------------------------------------------- |
| **Consistent**  | Uses **Raft consensus algorithm** to maintain consistency across nodes |
| **Distributed** | Multiple `etcd` members across control plane nodes                     |
| **Secure**      | Supports TLS for encrypted communication and authentication            |
| **Fast**        | Low-latency reads and writes for cluster state                         |
| **Persistent**  | Stores data on disk; survives reboots and restarts                     |

---

### ⚙️ How Kubernetes Uses `etcd`

1. `kubectl apply -f deployment.yaml`
2. API server receives the request and validates it
3. The desired state (Deployment) is **written to etcd**
4. Controllers and schedulers **read from etcd** to take action
5. Changes in cluster (e.g., Pod status) are **updated back to etcd**

---

### 🧠 How `etcd` Works (Internally)

* Follows the **Raft consensus protocol** to ensure that all copies of data on different nodes agree (leader + followers).
* One node is the **leader**; others are **followers**.
* All writes go through the leader, then propagated to followers.
* Only when a majority confirms, the change is committed.

## Install ETCD
   - Manual Setup & through Kubeadm: https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/05-ETCD-in-Kubernetes.md
   - install through package: https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/04-ETCD-For-Beginners.md

## Keypoint
- location of `etcd` mainfest: `/etc/kubernetes/manifests/etcd.yaml` 
- `etcd` by default listen on port 2379
- In kubeadm it run as a static pod manage by kubelet
- CLI tool for etcd: `etdctl` 
---
### Backup and Restore

* It is **critical to back up `etcd` regularly**, especially in production.
* You can use:

  ```bash
  etcdctl snapshot save backup.db
  etcdctl snapshot restore backup.db
  ```

---

### 🔐 Security Best Practices

* Enable **encryption at rest** (e.g., secrets stored in etcd).
* Use **TLS for communication** between API server and etcd.
* Restrict etcd access — don’t expose it publicly.
* Monitor for health using:

  ```bash
  etcdctl endpoint health
  ```

---

### 📁 Useful Commands (with `etcdctl`)

```bash
# Check health
etcdctl --endpoints=https://127.0.0.1:2379 endpoint health

# View keys (carefully!)
etcdctl get / --prefix --keys-only

# Backup
etcdctl snapshot save snapshot.db

# Restore
etcdctl snapshot restore snapshot.db
```

---

### Official Docs

* [etcd.io](https://etcd.io/)
* [K8s etcd documentation](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)




