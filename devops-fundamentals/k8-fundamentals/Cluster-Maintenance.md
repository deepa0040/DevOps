## Kubernetes Cluster Maintenance
Cluster maintenance ensures a Kubernetes environment remains healthy, secure, and performant. It involves monitoring, upgrades, resource management, and configuration changes.

#### 1. Node Maintenance

| Task               | Command/Method                                                         | Description                               |
| ------------------ | ---------------------------------------------------------------------- | ----------------------------------------- |
| Drain node         | `kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data` | Safely evict workloads before maintenance |
| Mark unschedulable | `kubectl cordon <node-name>`                                           | Prevent new pods from being scheduled     |
| Mark schedulable   | `kubectl uncordon <node-name>`                                         | Allow pods to be scheduled again          |
| Delete node        | `kubectl delete node <node-name>`                                      | Remove a node from cluster registry       |

#### Draining a Node
- For situations where the recovery time is uncertain, the safer method is to drain the node. Draining involves gracefully terminating the pods running on that node so they are recreated on other nodes.
- At the same time, draining marks the node as unschedulable (cordoned), preventing new pods from being scheduled on it until explicitly allowed.
- After the node's workloads have been safely relocated, you can reboot the node. When it comes back online, it remains unschedulable until you uncordon it, allowing new pods to be scheduled

#### Cordon vs. Drain
In addition to draining and uncordoning, Kubernetes provides the cordon command. Unlike drain, cordon only marks a node as unschedulable without terminating or relocating the currently running pods. This ensures that no new pods will be scheduled on the node.

        Be cautious when using cordon on a node with critical workloads. Since existing pods remain and new pods cannot be scheduled, your application might become overloaded or experience unexpected behavior.

## Cluster Upgrade
Upgrading a Kubernetes cluster is essential to keep up with security patches, new features, and bug fixes. It involves upgrading the control plane, node components, and sometimes add-ons.
### Upgrade Strategy
| Component       | Upgrade Order       |
| --------------- | ------------------- |
| Control Plane   | First               |
| Node Components | After control plane |
| Add-ons         | After nodes         |

### When to Upgrade?

Suppose you're running Kubernetes 1.10, and new releases 1.11 and 1.12 are available. Kubernetes officially supports up to the three most recent minor versions. With 1.12 as the latest, the supported versions are 1.12, 1.11, and 1.10. When version 1.13 is released, only 1.13, 1.12, and 1.11 will be supported. 

        It is advisable to upgrade your cluster to the next release before support for your current version is dropped.

        An effective upgrade strategy is to upgrade one minor version at a time (e.g., upgrade from 1.10 to 1.11, then from 1.11 to 1.12, and finally from 1.12 to 1.13) rather than attempting a large jump between versions. Keep in mind that the upgrade process may vary depending on your cluster setup. Managed Kubernetes services (such as Google Kubernetes Engine) offer a simple upgrade interface, while clusters deployed using tools like kubeadm or manual installation require more hands-on management.

### Pre-Upgrade Checklist
 - Backup etcd and cluster configuration

 - Drain and cordon nodes (for node upgrades)

 - Check deprecated APIs (kubectl get --raw /metrics)

 - Verify cluster health

 - Take snapshot of Persistent Volumes (if necessary)

 - Ensure compatibility between Kubernetes version & CNI, CSI, CRI
 
### Upgrade Process
The upgrade process generally involves two major steps:

- Upgrading the master nodes.
- Upgrading the worker nodes.

Master Node Upgrade:

- During upgrade, control plane components (such as the API server, scheduler, and controller managers) experience a brief interruption.
- Although management functionality (like kubectl commands or scaling deployments) is paused, the worker nodes continue to run and deliver applications.        
- However, keep in mind that if any pods fail during this period, they might not be restarted automatically.
- Once the master upgrade is complete, normal control plane operations resume.

After the master nodes are upgraded (for example, moving from version 1.10 to 1.11 while the worker nodes are still at 1.10), the next step is to upgrade the worker nodes.
```
# 1. Check current version
kubectl version

# 2. Plan upgrade
kubeadm upgrade plan

# 3. Apply upgrade
kubeadm upgrade apply v1.30.0

# 4. Update kubelet and kubectl
apt-get install -y kubelet=1.30.0-00 kubectl=1.30.0-00

# 5. Restart kubelet
systemctl daemon-reexec
systemctl restart kubelet

```

Worker Node upgrade:

There are several strategies for this:

- Upgrade all worker nodes simultaneously (which may result in downtime).
- pgrade one worker node at a time, allowing workloads to be shifted and ensuring continuous service.
- Add new nodes with the updated software version, migrate workloads to these new nodes, and then decommission the older nodes.
```
# 1. Cordon & drain node
kubectl cordon <node-name>
kubectl drain <node-name> --ignore-daemonsets

# 2. Upgrade kubeadm, kubelet & kubectl
apt-get install -y kubeadm=1.30.0-00 kubelet=1.30.0-00 kubectl=1.30.0-00

# 3. Join node upgrade
kubeadm upgrade node

# 4. Restart kubelet
systemctl daemon-reexec
systemctl restart kubelet

# 5. Uncordon node
kubectl uncordon <node-name>

```

### Upgrade Methods
| Tool      | Usage Type            | Description                            |
| --------- | --------------------- | -------------------------------------- |
| `kubeadm` | Self-managed cluster  | Upgrade control plane and worker nodes |
| `kOps`    | Self-managed on AWS   | Full cluster upgrade with automation   |
| `Rancher` | UI-driven management  | Upgrade via UI or CLI                  |
| `GKE`     | Managed cluster       | `gcloud container clusters upgrade`    |
| `EKS`     | Managed AWS cluster   | `eksctl upgrade cluster`               |
| `AKS`     | Managed Azure cluster | `az aks upgrade`                       |

Notes:

        It is important to note that not all components are required to run on the same version. Although different components can operate on varying release versions, the Kube API Server remains the primary control plane component that all others communicate with. Consequently, no component should ever run on a version higher than the API Server. For example:

        - The controller manager and scheduler may be one version lower than the API Server.
        - The Kubelet and Kube Proxy components may be two versions lower than the API Server.

        For instance, if the API Server is at version 1.10, then:

        - The controller manager and scheduler can run on version 1.10 or 1.9.
        - The Kubelet and Kube Proxy can run on version 1.8. Running any component on a version higher than the API Server (e.g., 1.11 when the API Server is 1.10) is not recommended.



## Kubernetes Backup and Restore Methods

Backups are critical to ensure **disaster recovery**, **rollback capabilities**, and **data integrity** in Kubernetes environments. Backups typically involve:

* ⏳ **etcd (cluster state)**
* 📁 **Persistent volumes (application data)**
* 📦 **Kubernetes resources (YAML configs, secrets, etc.)**

---

### 1. **etcd Backup & Restore (for Self-Managed Clusters)**

Reference: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#built-in-snapshot

etcd is the **key-value store** for all Kubernetes cluster data.

#### ✅ Backup etcd 

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

> Output: `snapshot.db` file, contains all cluster metadata.

#### 🔁 Restore etcd

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

After restoring, update `etcd.yaml` to point to the new data directory.

---

### 2. **Kubernetes Resource Backups (YAML/Manifest)**

#### ✅ Backup (Manually)

```bash
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml
kubectl get configmaps,secrets,pvc,ingress --all-namespaces -o yaml >> cluster-backup.yaml
```

#### 🔁 Restore

```bash
kubectl apply -f cluster-backup.yaml
```

---

### 3. **Volume Data Backup (Persistent Volumes)**

Depends on the **StorageClass/CSI plugin** used. Options:

| Method       | Notes                                                        |
| ------------ | ------------------------------------------------------------ |
| Snapshot API | Use `VolumeSnapshot` via CSI drivers (e.g. EBS, GCE, Cinder) |
| Rsync/Copy   | Mount PVCs and copy files manually or via init container     |
| Restic       | File-level backup tool (used with Velero)                    |
| Cloud-native | AWS Backup, Azure Recovery Vault, GCP Snapshots              |

---

### 4. **Using Velero (Popular Tool)**

Velero can back up **Kubernetes resources**, **volumes**, and supports **scheduling**, **hooks**, and **cloud integration**.

#### ✅ Backup

```bash
velero backup create daily-backup --include-namespaces default
```

#### 🔁 Restore

```bash
velero restore create --from-backup daily-backup
```

#### Provider Support

* AWS S3, Azure Blob, GCP Storage
* Use `restic` with Velero for PV backup

---

### 5. **Other Backup Tools**

| Tool                  | Features                                       |
| --------------------- | ---------------------------------------------- |
| **Velero**            | Complete solution for resource + PV backup     |
| **Kasten K10**        | Enterprise backup with UI, RBAC, multi-cluster |
| **Stash by AppsCode** | CRD-based backup/restore for apps & volumes    |
| **Kubegres**          | Stateful PostgreSQL + backup inside K8s        |

---

### 6. **Best Practices**

| Practice                           | Why It's Important                     |
| ---------------------------------- | -------------------------------------- |
| Automate scheduled backups         | Avoid manual error, ensure consistency |
| Store backups offsite              | Protect from node or disk failure      |
| Encrypt sensitive backup data      | Security and compliance                |
| Test restore regularly             | Validate DR plan is functional         |
| Use version control for YAML files | Track config changes over time         |


