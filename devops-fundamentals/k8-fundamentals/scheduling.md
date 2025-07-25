## Scheduling 
The scheduler is a control plane component that watches for unscheduled Pods and assigns them to suitable Nodes based on constraints and available resources.

    Think of the kube-scheduler as the brain that decides where each pod should run.
    Manifest Path	: /etc/kubernetes/manifests/kube-scheduler.yaml
    Pod Type: Static pod (in kubeadm)

### Key Responsibilities of the Scheduler
| Responsibility          | Description                                      |
| ----------------------- | ------------------------------------------------ |
| **Pod placement**       | Assigns Pods to the most appropriate Node        |
| **Resource awareness**  | Considers CPU, memory, disk, etc.                |
| **Constraint matching** | Honors rules like nodeSelector, affinity, taints |
| **Scoring**             | Ranks Nodes to choose the best fit               |
| **Extensibility**       | Allows custom scheduler plugins & policies       |

### Scheduler Flow
- A new Pod is created (by user or controller)

- Pod is unscheduled (Pending phase)

- kube-scheduler watches for pending Pods

- Evaluates all Nodes:

       - Filters (based on rules, resources)

       - Scores (ranks best matches)

- Assigns Pod to a Node (via API Server)

- Kubelet on Node creates the Pod

## How Scheduling Works
What do you do when you do not have a scheduler in your cluster?
- Every POD has a field called NodeName that by default is not set. You don't typically specify this field when you create the manifest file, kubernetes adds it automatically.
- Once identified it schedules the POD on the node by setting the nodeName property to the name of the node by creating a binding object.

Example:
```yaml
spec:
 containers:
 - name: nginx
   image: nginx
   ports:
   - containerPort: 8080
 nodeName: node02
```

### Manual Scheduling: (No scheduler)
- You can manually assign pods to node itself. Well without a scheduler, to schedule pod is to set nodeName property in your pod definition file while creating a pod.
- Another way is binding
```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
  name: nginx
spec:
 containers:
 - name: nginx
   image: nginx
   ports:
   - containerPort: 8080
```

### Labels and Selectors
Labels are key-value pairs attached to Kubernetes objects like Pods, Services, Nodes, etc.

    They help identify, group, and organize resources based on meaningful attributes (e.g., environment, app, version).

Example: 

```yaml
metadata:
  labels:
    app: nginx
    env: production
    version: v1
```
Selectors allow you to filter or match resources based on their labels.

```yaml
selector:
  matchExpressions:
    - key: app
      operator: In
      values: ["nginx", "apache"]
    - key: env
      operator: NotIn
      values: ["prod"]
```
| Operator       | Description                |
| -------------- | -------------------------- |
| `In`           | Value exists in list       |
| `NotIn`        | Value is **not** in list   |
| `Exists`       | Key is present (any value) |
| `DoesNotExist` | Key is missing             |

Example:

```yaml
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
### Labels and Selectors – Command Table

| **Task**                            | **Command**                                             |
| ----------------------------------- | ------------------------------------------------------- |
| View all labels on pods             | `kubectl get pods --show-labels`                        |
| View labels on a specific pod       | `kubectl get pod <pod-name> --show-labels`              |
| Add a label to a resource           | `kubectl label pod <pod-name> env=dev`                  |
| Add multiple labels                 | `kubectl label pod <pod-name> team=frontend version=v1` |
| Change a label value                | `kubectl label pod <pod-name> env=prod --overwrite`     |
| Remove a label                      | `kubectl label pod <pod-name> env-`                     |
| Filter resources by label           | `kubectl get pods -l app=nginx`                         |
| Filter with multiple labels         | `kubectl get pods -l app=nginx,env=prod`                |
| Use inequality label selector       | `kubectl get pods -l 'env!=production'`                 |
| Select pods with `matchExpressions` | (Used in YAML - e.g., ReplicaSet or Deployment)         |
| Delete resources by label           | `kubectl delete pod -l app=nginx`                       |
| List services selecting a label     | `kubectl get svc --selector app=nginx`                  |

### Annotations
Annotations differ from labels and selectors in that they are used to store additional metadata that is not intended for selection. This metadata might include details such as tool versions, build information, or contact information.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  annotations:
    buildversion: "1.34"
```

## Taints and Tolerations
Taints and Tolerations are used to set restrictions on what pods can be scheduled on a node.
- A taint is a way to repel Pods from being scheduled on a Node unless they explicitly tolerate it.

    Taints are applied to nodes to control what pods can run on them.


- A toleration is applied to a Pod. It tells Kubernetes, 👉 “This pod can run on nodes with matching taints.”

### Why Use Taints & Tolerations?
| Use Case                             | Example                                                         |
| ------------------------------------ | --------------------------------------------------------------- |
| Isolate nodes for specific workloads | Run only monitoring or GPU pods                                 |
| Protect system nodes                 | Prevent regular pods from running on master/control-plane nodes |
| Enforce pod-node affinity            | Allow only authorized pods to run on tainted nodes              |

### 🔧 Taint Syntax (Node-level)
```bash
kubectl taint nodes <node-name> key=value:effect
example:kubectl taint nodes node1 env=prod:NoSchedule
```
### 🎫 Toleration Syntax (Pod-level YAML)
```yml
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "prod"
      effect: "NoSchedule"
```
## Taint Effects
| Effect             | Behavior                                                      |
| ------------------ | ------------------------------------------------------------- |
| `NoSchedule`       | Pod **won’t be scheduled** unless it tolerates the taint      |
| `PreferNoSchedule` | Scheduler tries to avoid the node, but **may still schedule** |
| `NoExecute`        | **Evicts running pods** that don’t tolerate it                |
### Commands Reference
| Task                  | Command                                      |                        |
| --------------------- | -------------------------------------------- | ---------------------- |
| View taints on a node | `kubectl describe node <node-name>`          |                        |
| Add a taint           | `kubectl taint nodes node1 key=value:effect` |                        |
| Remove a taint        | `kubectl taint nodes node1 key:effect-`      |                        |
| Check pod tolerations | `kubectl get pod <pod> -o yaml` |

## Summary Table

| Concept         | Scope                                   | Purpose                                            |
| --------------- | --------------------------------------- | -------------------------------------------------- |
| **Taint**       | Node                                    | Prevent pods from being scheduled unless tolerated |
| **Toleration**  | Pod                                     | Allow pod to run on tainted node                   |
| **Key Effects** | NoSchedule, PreferNoSchedule, NoExecute |                                                    |

## Node Selectors
nodeSelector is the simplest way to constrain a Pod to run only on nodes with a specific label.

    It’s a key-value pair match between a pod’s requirement and node’s labels.

### How It Works
- Nodes have labels (e.g., disktype=ssd, zone=us-east)

- Pods specify a nodeSelector field in their spec

- Scheduler only assigns the pod to nodes with matching labels

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 nodeSelector:
  size: Large
  ```
### When to Use nodeSelector
| Use Case                     | Description                                            |
| ---------------------------- | ------------------------------------------------------ |
| Scheduling on specific nodes | e.g., Only nodes with GPUs, SSDs, or in a certain zone |
| Simple constraints           | Basic use cases without complex logic                  |
| Reserved workloads           | Ensure logging or monitoring pods go to special nodes  |

### Common kubectl Commands
| Task                        | Command                                       |
| --------------------------- | --------------------------------------------- |
| List node labels            | `kubectl get nodes --show-labels`             |
| Add label to node           | `kubectl label node <node-name> disktype=ssd` |
| Remove label from node      | `kubectl label node <node-name> disktype-`    |
| Describe node (view labels) | `kubectl describe node <node-name>`         |

### Node Selector - Limitations  
- We used a single label and selector to achieve our goal here. But what if our requirement is much more complex.
For Example: large or medium label node.
For this we have Node Affinity and Anti Affinity

### ⚠️ Notes
- With Node Selectors we cannot provide the advance expressions.

- nodeSelector does exact match only. No conditions like In, NotIn, etc.

- If no node has the matching label, the pod stays in Pending.

- For advanced logic, use nodeAffinity instead.

### Node Affinity
Node Affinity is a way to control which nodes a pod can be scheduled on, based on node labels. It's part of Kubernetes' scheduling constraints.

### Why Use Node Affinity?
- Run certain workloads only on specific nodes (e.g., nodes with GPUs, SSDs, certain regions).

- Enforce soft or hard placement rules using labels.

Example: 
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
                  - Medium

```

### Types of Node Affinity:
| Type  | Affinity Key                                                   | Scheduling Rule | Execution Rule | Description                                                                                                                                                                                                                           |
| ----- | -------------------------------------------------------------- | --------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | `requiredDuringSchedulingIgnoredDuringExecution`               | ✅ Required      | ❌ Ignored      | The scheduler **only places the pod on a node that matches** the affinity rules. If no matching node exists, **the pod won't be scheduled at all**. After it's running, the rule is not checked again—even if the node label changes. |
| **2** | `preferredDuringSchedulingIgnoredDuringExecution`              | ✅ Preferred     | ❌ Ignored      | The scheduler **prefers nodes that meet the affinity rules**, but will place the pod on another node **if no matching nodes are available**. Like Type 1, this is ignored after scheduling.                                           |
| **3** | `requiredDuringSchedulingRequiredDuringExecution` *(planned)*  | ✅ Required      | ✅ Required     | The pod is scheduled **only on matching nodes**, and if the node **later stops matching**, the pod will be **evicted or rescheduled**. This ensures the pod always runs on a node that satisfies the affinity rule.                   |
| **4** | `preferredDuringSchedulingRequiredDuringExecution` *(planned)* | ✅ Preferred     | ✅ Required     | The scheduler tries to place the pod on a matching node if possible, but will still schedule it on others. However, **once running**, if the node **does not match the rule**, the pod will be **evicted**.                           |

### nodeSelector vs nodeAffinity
| Feature          | `nodeSelector` | `nodeAffinity`                        |
| ---------------- | -------------- | ------------------------------------- |
| Simple equality  | ✅ Yes          | ✅ Yes                                 |
| Expressions      | ❌ No           | ✅ Yes (`In`, `NotIn`, `Exists`, etc.) |
| Soft constraints | ❌ No           | ✅ Yes (preferred type)                |


### Taints and Tolerations vs Node Affinity

| Feature                    | **Taints & Tolerations**                                                              | **Node Affinity**                                                  |
| -------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 🧠 **Purpose**             | **Prevent pods** from being scheduled on certain nodes unless they tolerate the taint | **Attract pods** to nodes based on matching rules                  |
| 🔁 **Control Type**        | **Exclusion / Blocking** mechanism                                                    | **Selection / Matching** mechanism                                 |
| 💡 **Used When**           | You want to **repel pods** from certain nodes (e.g., dedicated, critical workloads)   | You want to **target pods** to certain nodes                       |
| 🧷 **Defined On**          | **Taints** → on **nodes**<br>**Tolerations** → on **pods**                            | **Node labels** → on **nodes**<br>**Affinity rules** → on **pods** |
| 🚫 **Effect**              | Node rejects all pods **without tolerations** for its taints                          | Scheduler just **prefers or requires** nodes for pods              |
| ✅ **Soft / Hard Matching** | No soft option; either tolerated or not                                               | Has both **required** and **preferred** modes                      |
| 🧰 **Typical Use Case**    | Keep non-critical pods off production or GPU nodes                                    | Place specific workloads on labeled zones (e.g., region, type)     |

## Limitation:
#### Taints & Tolerations
- Taints do not prevent a pod from being scheduled on other nodes—they only stop pods without matching tolerations from being scheduled on tainted nodes.

- This means pods can still be placed on any untainted node, even if it's not ideal for workload isolation.

#### Node Affinity
- Node Affinity controls which nodes a pod can be scheduled on by matching labels.

- However, it does not prevent other pods from being scheduled on that node unless those other pods have their own placement restrictions.

- So, even if a node is intended for a specific workload, other unrestricted pods may still be scheduled on it.

#### Combining Taints and Tolerations with Node Affinity
For exclusive node usage, combining both strategies is the optimal solution. The integration works as follows:

- Apply taints on nodes and specify corresponding tolerations in pod configurations to block any pod without the proper toleration.
- Use node affinity rules to ensure that each pod is only scheduled on a node with a matching label.

This combined approach dedicates the nodes exclusively to the intended pods, assuring correct pod assignments and preventing interference by other workloads.

### Why Combine Them?
| Feature                        | Taints & Tolerations | Node Affinity          | Together |
| ------------------------------ | -------------------- | ---------------------- | -------- |
| Prevents unwanted pods         | ✅ (via taints)       | ❌                      | ✅        |
| Ensures pod goes to right node | ❌                    | ✅ (via label matching) | ✅        |
| Guarantees exclusive access    | ❌                    | ❌                      | ✅        |


## Resource Limit
Kubernetes lets you control how much CPU and memory (RAM) a pod or container can request and use, via resource requests and limits.

### Two Key Settings

| Type        | Description                                                                                         |
| ----------- | --------------------------------------------------------------------------------------------------- |
| **Request** | The **minimum** amount of CPU/memory a container is **guaranteed to get**. Used for **scheduling**. |
| **Limit**   | The **maximum** amount of CPU/memory the container is **allowed to use**. Prevents overuse.         |

### Why Use Them?
- Prevent one pod from consuming too many resources

- Ensure fair distribution across containers

- Avoid OOM Killed (out of memory) or node overloads

- Help the scheduler make better placement decisions

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"     # 0.25 CPU core
      limits:
        memory: "256Mi"
        cpu: "500m"     # 0.5 CPU core
```
Limit across Namespace
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: dev
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "1"
    defaultRequest:
      memory: "256Mi"
      cpu: "500m"
    type: Container

```
### Behavior Explained
| Case                                     | What Happens                                  |
| ---------------------------------------- | --------------------------------------------- |
| CPU exceeds **limit**                    | Container is **throttled**, not killed.       |
| Memory exceeds **limit**                 | Container is **terminated** with `OOM (Out Of Memory) Killed`. |
| Node doesn't have enough for **request** | Pod **won’t be scheduled** there.             |

### Note
Keep in mind that changes to a LimitRange affect only new pods created after the LimitRange is applied or updated.

## DaemonSets
A DaemonSet ensures that a copy of a specific pod runs on all (or some) nodes in a Kubernetes cluster.

  DaemonSets are like replicasets, as it helps in to deploy multiple instances of pod. But it runs one copy of your pod on each node in your cluster.

Example:  kube-proxy, which Kubernetes requires on all worker nodes.

### Use case
| Use Case                              | Example Tools                           |
| ------------------------------------- | --------------------------------------- |
| Log collection                        | Fluentd, Filebeat, Logstash             |
| Monitoring agents                     | Prometheus Node Exporter, Datadog agent |
| Security agents                       | Falco, OSSEC                            |
| Storage drivers or node-level daemons | CSI drivers, NFS client pods            |

### How It Works
- A DaemonSet automatically adds a pod to every new node.

- When a node is removed, the pod is deleted from that node.

- You can limit which nodes get the pod using node selectors, affinity, or taints/tolerations.

Example:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
        - name: monitoring-agent
          image: monitoring-agent

```

## Static Pods
Static Pods are pods that are managed directly by the kubelet, not by the Kubernetes Control Plane (like the scheduler or controller manager).

They are ideal for running core components (e.g., kube-apiserver, etcd, kube-controller-manager) on specific nodes.

### Key Characteristics
| Feature                               | Description                                                                                    |
| ------------------------------------- | ---------------------------------------------------------------------------------------------- |
| ❌ Not managed by API Server           | They don’t go through the scheduler or controllers.                                            |
| ✅ Managed by Kubelet only             | Each node’s kubelet watches for static pod files and starts them directly.                     |
| 🗂️ Stored in a manifest file         | YAML files placed in a **specific directory** (usually `/etc/kubernetes/manifests/`).          |
| 📦 Automatically becomes a Mirror Pod | A "mirror pod" is created on the API server to show the pod status, but **you can't edit it**. |
| 🔁 Auto-restart on crash              | The kubelet will restart static pods if they fail.                                             |

### How Are They Used in Practice?
Kubeadm uses static pods to run Kubernetes control plane components (kube-apiserver, controller-manager, scheduler, etcd) on the master node.

They are required to bootstrap the cluster even before the API server is ready.

###  Static Pod vs Regular Pod
| Feature         | **Static Pod**                       | **Regular Pod**                  |
| --------------- | ------------------------------------ | -------------------------------- |
| Managed by      | Kubelet (on each node)               | Kubernetes control plane         |
| Defined in      | File on disk (`/etc/...`)            | API server (via `kubectl`)       |
| Visible in API? | Yes, as **Mirror Pod** (read-only)   | Yes, full lifecycle control      |
| Uses scheduler? | ❌ No                                 | ✅ Yes                            |
| Common use case | Cluster bootstrap, node-level agents | Apps and services managed by K8s |

### Notes
- You can't delete or edit a static pod using kubectl delete—you must edit or remove the file from the manifest path.
- The kubelet can create both kinds of pods - the static pods and the ones from the api server at the same time.

### Static Pods vs DaemonSets
| **Static PODs**                                    | **DaemonSets**                                            |
| -------------------------------------------------- | --------------------------------------------------------- |
| Created by the **Kubelet**                         | Created by **Kube-API server** (DaemonSet Controller)     |
| Deploy **Control Plane components** as Static Pods | Deploy **Monitoring Agents**, **Logging Agents** on nodes |
|**Ignored by the Kube-Scheduler**                                                     | **Ignored by the Kube-Scheduler**                         |

### When to Use Which
| Scenario                                | Use Static Pod? | Use DaemonSet? | Notes                                                 |
| --------------------------------------- | --------------- | -------------- | ----------------------------------------------------- |
| Deploy core components on control plane | ✅ Yes           | ❌ No           | Used by `kubeadm` for `etcd`, `kube-apiserver`, etc.  |
| Run logging/monitoring agents           | ❌ No            | ✅ Yes          | DaemonSet automates multi-node deployment             |
| No access to Kubernetes API (bootstrap) | ✅ Yes           | ❌ No           | Static Pods run even before control plane is fully up |
| Need auto-scaling, rolling update       | ❌ No            | ✅ Yes          | DaemonSet supports standard controller features       |
| Testing or debugging a node directly    | ✅ Yes           | ❌ No           | Quickly place a pod on a node without API interaction |

## Priority Classes
Priority Classes are Kubernetes objects that define the importance (priority value) of Pods. They influence Pod scheduling and preemption during resource contention.
### Key Concepts
| Term            | Meaning                                                                    |
| --------------- | -------------------------------------------------------------------------- |
| `PriorityClass` | A Kubernetes object that defines a `value` and optional `globalDefault`    |
| `value`         | Integer representing the priority (higher = more important)                |
| `preemption`    | Higher-priority Pods can **evict lower-priority Pods** when nodes are full |
| `preemptionPolicy`    | it's used with Priority Classes to control whether a pod is allowed to preempt (evict) other pods when scheduling. |
| `globalDefault` | If true, it's the default class if a Pod doesn't specify one               |
| `priorityClassName` | Assign to pod by specifying the field in pod template sepc    |

### Values for preemptionPolicy
| Value                            | Meaning                                                               |
| -------------------------------- | --------------------------------------------------------------------- |
| `PreemptLowerPriority` (default) | Pod **can preempt** lower-priority pods to get scheduled              |
| `Never`                          | Pod **will NOT preempt** any other pod, even if it can't be scheduled |

### How Priority Classes Work
- Scheduler looks at priority when deciding which Pods to run first.

- If the cluster runs out of resources, lower-priority Pods may be preempted (evicted) to make space for higher-priority Pods.

- Priority affects:

      Scheduling
      Preemption
      Not QoS class (which is based on CPU/memory requests/limits)

Example:
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "This priority class is for critical workloads"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
globalDefault: true
description: "This is the default priority class"
```
#### Assign to a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  priorityClassName: high-priority
  preemptionPolicy: Never
  containers:
    - name: app
      image: nginx
```

### Preemption in Action
When a high-priority Pod can't be scheduled due to resource shortage:

- Scheduler looks for lower-priority Pods on nodes.

- Those Pods may be terminated (preempted) to free up resources.
### Limitations
| Limitation                      | Details                                                                 |
| ------------------------------- | ----------------------------------------------------------------------- |
| Only works for **pending** Pods | Priority is only considered when Pod is waiting to be scheduled         |
| Can cause **Pod churn**         | Preemptions can cause restarts and instability if not handled carefully |
| Doesn't affect running order    | Only affects scheduling, not container start order in a Pod             |

### When to Use
- Critical system components (e.g., monitoring, logging)

- Workloads that must always run

- Tiered services (e.g., Premium vs Free user services)

- Multi-tenant clusters

### Commands
| 🏷️ Action                   | 💻 Command                                     | 📝 Description                                     |
| ---------------------------- | ---------------------------------------------- | -------------------------------------------------- |
| List all priority classes    | `kubectl get priorityclass`                    | Show existing priority classes                     |
| Describe a priority class    | `kubectl describe priorityclass <name>`        | View value, preemption, global default             |
| Create/apply priority class  | `kubectl apply -f priorityclass.yaml`          | Define priority level for pods                     |
| Assign priority to a Pod     | Add `priorityClassName: high-priority` in YAML | Ensures the Pod has higher scheduling precedence   |
| Disable preemption for a Pod | Add `preemptionPolicy: Never` in Pod spec      | Prevents the Pod from evicting lower-priority Pods |


## Multiple Schedulers
### What Are Schedulers in Kubernetes?
The Kubernetes scheduler is a control plane component that assigns Pods to Nodes based on resource availability, policies, constraints, and priorities.

By default, Kubernetes uses the default scheduler (kube-scheduler).

### What Are Multiple Schedulers?
You can run more than one scheduler in a cluster — including your own custom scheduler — and assign specific Pods to be scheduled by a specific scheduler.

### Why Use Multiple Schedulers?
| Use Case                                | Explanation                                                        |
| --------------------------------------- | ------------------------------------------------------------------ |
| Custom scheduling logic                 | You want Pods to follow rules not handled by the default scheduler |
| Multi-tenant or multi-team environments | Each team uses its own scheduler logic                             |
| Testing new algorithms                  | Evaluate custom scheduling without affecting production            |
| Scheduling batch vs real-time workloads | Use different schedulers for different workload types              |

### How It Works
You can run multiple scheduler binaries, each with a unique name and configuration.

#### Step-by-Step:
- Deploy an additional scheduler (e.g., as a Deployment in the kube-system namespace)

- Give it a different name, e.g., "my-scheduler"

- Use schedulerName in your Pod spec to assign it:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: custom-scheduled-pod
spec:
  schedulerName: my-scheduler
  containers:
    - name: nginx
      image: nginx
```
Kubernetes will NOT assign this Pod to any node unless a scheduler named my-scheduler picks it up.

### Configuring Schedulers with YAML
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
  ```
### Deploying an Additional Scheduler
You can deploy an additional scheduler using the existing kube-scheduler binary, tailoring its configuration through specific service files.

#### Step 1: Download the kube-scheduler Binary
```
wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler
```
#### Step 2: Create Service Files
```
# kube-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler --config=/etc/kubernetes/config/kube-scheduler.yaml
```
#### Step 3: Define Scheduler Configuration Files
```
# my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```
### Deploying the Custom Scheduler as a Pod

#### Pod Definition
```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
    - name: kube-scheduler
      image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      command:
        - kube-scheduler
        - --address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --config=/etc/kubernetes/my-scheduler-config.yaml
```
The corresponding custom scheduler configuration file might look like:
```
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

### Configuring Workloads to Use the Custom Scheduler
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  schedulerName: my-custom-scheduler
```
### Commands
| 🏷️ Action                               | 💻 Command                                                                                                                                          | 📝 Description                               |                                   |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | --------------------------------- |
| List schedulers in cluster               | \`kubectl get pods -n kube-system                                                                                                                   | grep scheduler\`                             | See default/custom scheduler pods |
| View logs of a scheduler                 | `kubectl logs -n kube-system <scheduler-pod-name>`                                                                                                  | Debug scheduler activity                     |                                   |
| Assign a Pod to a scheduler              | Add `schedulerName: my-scheduler` in Pod YAML and `kubectl apply -f pod.yaml`                                                                       | Use a custom scheduler for specific Pods     |                                   |
| List unscheduled (Pending) Pods          | `kubectl get pods --field-selector=status.phase=Pending`                                                                                            | Shows Pods waiting for scheduling                                                 
| Run custom scheduler for testing         | `kubectl run my-scheduler --image=k8s.gcr.io/kube-scheduler:v1.28.0 --command -- kube-scheduler --scheduler-name=my-scheduler --leader-elect=false` | Launch a test custom scheduler pod           |                                   |
| Delete custom scheduler                  | `kubectl delete pod my-scheduler` or `kubectl delete -f custom-scheduler.yaml`                                                                      | Remove test/custom scheduler                 |                                   |
| View default scheduler config            | `kubectl get configmap -n kube-system kube-scheduler -o yaml`                                                                                       | View scheduler settings (if using ConfigMap) |                                   |

### Event Commands
| 🏷️ Action                          | 💻 Command                                                       | 📝 Description                                               |                                    |
| ----------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------- |
| View events in current namespace    | `kubectl get events`                                             | List recent events                                           |                                    |
| View events in all namespaces       | `kubectl get events -A`                                          | List events across entire cluster                            |                                    |
| View detailed event info            | `kubectl get events -o wide`                                     | Show Source, Count, Subobject, etc.                          |                                    |
|

### Configuring Scheduler Profiles
Scheduler profiles allow you to run multiple scheduling configurations using a single kube-scheduler binary, each with different rules/plugins.

Each profile can target a different type of workload (e.g., batch jobs vs latency-sensitive apps), and you assign a profile to a Pod using schedulerName.

    No profile = default scheduler is used.
### How It Works
- You define multiple profiles in the kube-scheduler configuration file.

- Each profile is given a unique schedulerName.

- Pods can request a specific profile by setting schedulerName: <name>.
### Sample kube-scheduler Config (with Profiles)
```yml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
    plugins:
      score:
        enabled:
          - name: NodeResourcesBalancedAllocation

  - schedulerName: batch-scheduler
    plugins:
      score:
        enabled:
          - name: NodeResourcesLeastAllocated

```
#### Pod Using a Custom Profile
```yml
apiVersion: v1
kind: Pod
metadata:
  name: batch-job
spec:
  schedulerName: batch-scheduler
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]

```
### Benefits of Profiles (vs multiple schedulers)
| Feature                            | Scheduler Profiles | Multiple Schedulers          |
| ---------------------------------- | ------------------ | ---------------------------- |
| Performance overhead               | ✅ Low (one binary) | ❌ Higher (multiple binaries) |
| Easier to manage                   | ✅ Yes              | ❌ Needs separate pods/logs   |
| Supports custom plugins            | ✅ Per-profile      | ✅ Per scheduler              |
| Pod selection with `schedulerName` | ✅ Supported        | ✅ Supported                  |
