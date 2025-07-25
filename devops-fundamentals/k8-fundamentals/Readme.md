## ☸️ Kubernetes Architecture (Core Concepts)

Kubernetes follows a **master-worker (control plane - node)** architecture. It's designed to **orchestrate** containerized applications across a cluster of machines efficiently.

```
                 +-------------------------------+
                 |       Control Plane (Master)  |
                 +-------------------------------+
                         |            |          
       +----------------+            +----------------+
       |                                         |
+---------------+                       +---------------+
|  Worker Node 1|                       | Worker Node 2 |
+---------------+                       +---------------+
```

---

##  1. Control Plane (Master Node)

Responsible for managing the entire Kubernetes cluster.

| Component                    | Description                                                                            |
| ---------------------------- | -------------------------------------------------------------------------------------- |
| **kube-apiserver**           | Frontend of the control plane. Accepts REST requests (kubectl, dashboard, etc.)        |
| **etcd**                     | Key-value store used to store all cluster data (config, state, secrets)                |
| **kube-scheduler**           | Assigns newly created pods to suitable nodes based on resources, affinity rules, etc.  |
| **kube-controller-manager**  | Runs controllers (replica, endpoint, job, etc.) to ensure desired state matches actual |
| **cloud-controller-manager** | (Optional) Connects to cloud-specific APIs like AWS, GCP                               |

---

## 2. Worker Node (Minion)

Runs the actual applications in **Pods**.

| Component             | Description                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------ |
| **kubelet**           | Agent that runs on each node, communicates with API server, ensures containers are running |
| **kube-proxy**        | Maintains network rules for Pod communication & service discovery                          |
| **container runtime** | Software to run containers (e.g., containerd, Docker, CRI-O)                               |

---

## 🔁 Cluster Flow Example

1. **kubectl apply -f deployment.yaml**
2. **kube-apiserver** receives the request and validates it
3. **etcd** stores the desired state
4. **controller-manager** creates ReplicaSet, which creates Pod
5. **scheduler** assigns the Pod to a node
6. **kubelet** on that node runs the Pod via **container runtime**
7. **kube-proxy** sets up networking so the Pod can be accessed

---

## 📌 Key Concepts

| Term           | Meaning                                                  |
| -------------- | -------------------------------------------------------- |
| **Node**       | A machine (VM or physical) in the cluster                |
| **Pod**        | The smallest unit in K8s, wraps one or more containers   |
| **ReplicaSet** | Ensures a specified number of Pod replicas are running   |
| **Deployment** | Manages ReplicaSets and provides rolling updates         |
| **Service**    | Exposes Pods internally or externally                    |
| **Namespace**  | Virtual clusters inside the main cluster (for isolation) |

---

## ETCD



## `etcd` – Kubernetes’ Key-Value Store

---

### What is `etcd`?

* `etcd` is a **distributed**, **consistent**, and **highly available** **key-value store**.
* It's the **source of truth** for Kubernetes — it stores all **cluster state and configuration data**.
* Think of it as the **brain** of your Kubernetes cluster.

---

### What Does `etcd` Store?

| Data Type                  | Example                       |
| -------------------------- | ----------------------------- |
| Cluster state              | Node info, health, roles      |
| Pod/Deployment definitions | All YAML manifest data        |
| Secrets & ConfigMaps       | Secure and config values      |
| Resource quotas            | CPU/memory limits             |
| Role-based access control  | RBAC policies                 |
| Service discovery info     | Internal DNS and service maps |

---

### Example: How Kubernetes Uses `etcd`

1. `kubectl apply -f deployment.yaml`
2. API server receives the request and validates it
3. The desired state (Deployment) is **written to etcd**
4. Controllers and schedulers **read from etcd** to take action
5. Changes in cluster (e.g., Pod status) are **updated back to etcd**


## Keypoint
- location of `etcd` mainfest: `/etc/kubernetes/manifests/etcd.yaml` 
- `etcd` by default listen on port 2379
- In kubeadm it run as a static pod manage by kubelet
- CLI tool for etcd: `etdctl` 

---
## Kube-apiserver
- Kube-apiserver is the primary component in kubernetes.
- Kube-apiserver is responsible for `authenticating`, `validating requests`, `retrieving` and `Updating data` in ETCD key-value store. In fact kube-apiserver is the only component that interacts directly to the etcd datastore. The other components such as kube-scheduler, kube-controller-manager and kubelet uses the API-Server to update in the cluster in their respective areas.

## Kube Controller Manager
Kube Controller Manager manages various controllers in kubernetes.
- In kubernetes terms, a controller is a process that continuously monitors the state of the components within the system and works towards bringing the whole system to the desired functioning state.
- Example:

Node Controller: Responsible for monitoring the state of the Nodes and taking necessary actions to keep the application running.

Replication Controller: It is responsible for monitoring the status of replicasets and ensuring that the desired number of pods are available at all time within the set.

Other Controllers: There are many more such controllers available within kubernetes like Deployment, Service etc

## Kube Scheduler
kube-scheduler is responsible for scheduling pods on nodes.

- The kube-scheduler is only responsible for deciding which pod goes on which node. It doesn't actually place the pod on the nodes, that's the job of the kubelet.

Why do you need a Scheduler?
- Filter nodes based on desired resource requirement like cpu and memory
- Rank the nodes based on free amount of free resources after placing the pod.   

## Kubelet
Kubelet is the sole point of contact for the kubernetes cluster

- The kubelet will create the pods on the nodes, the scheduler only decides which pods goes where.

## Kube Proxy
Within Kubernetes Cluster, every pod can reach every other pod. This is accomplished by deploying a pod networking cluster to the cluster.
- Kube-Proxy is a process that runs on each node in the kubernetes cluster.
- It is responsible for routing network traffic between services and pods.
- In kubeadm clusters, the kube-proxy runs as a DaemonSet (not a static pod):

### 🔗 How It Works
When a Service is created:

- API Server stores it in etcd

- kube-proxy watches the API server

- It updates local node’s iptables or IPVS rules

- Incoming traffic is forwarded to the appropriate pod behind the service

### Key Responsibilities

| Responsibility              | Description                                                                          |
| --------------------------- | ------------------------------------------------------------------------------------ |
| **Service-to-Pod routing**  | Routes traffic to backend Pods behind a ClusterIP, NodePort, or LoadBalancer service |
| **Maintains iptables/ipvs** | Manages rules for routing packets using either `iptables` or `IPVS`                  |
| **Load Balancing**          | Balances traffic across multiple backend pods                                        |
| **Dynamic updates**         | Watches for service or pod changes via API server and updates rules dynamically      |


### ⚙️ Modes of Operation
| Mode          | Description                                                            |
| ------------- | ---------------------------------------------------------------------- |
| **iptables**  | Default in most setups. Uses Linux firewall rules to route traffic     |
| **ipvs**      | Offers better performance at scale. Uses Linux IPVS for load balancing |
| **userspace** | Deprecated/legacy mode. Routes traffic through a user process (slow)   |

## Pods
Kubernetes doesn't deploy containers directly on the worker node. The Container are encapsulated into a kubernetes object known as pods.
- A pod is a single instance of an application.
- A pod is the smallest object that you can create in K8's.
- To scale your application we launch new pod i.e instance of the same application.
- Pod will have a one-to-one relationship with containers running your application.
- A single pod can have multiple containers except for the fact that they are usually not multiple containers of the same kind.

## ReplicaSets
A ReplicaSet ensures that a specified number of identical Pod replicas are always running in a Kubernetes cluster.
-  It replaces Pods if they crash, get deleted, or fail health checks.
- Use case: To maintain a consistent number of running Pods based on a label selector.

### How ReplicaSet Works
- You define the desired number of replicas (e.g., 3).

- ReplicaSet ensures 3 matching pods are running at all times.

- If a pod crashes or is deleted, it immediately creates a new one.

- If extra matching pods are found, it terminates the excess.


### Labels and Selectors: 
Labels in Kubernetes are critical because they enable controllers, such as ReplicaSets, to identify and manage the appropriate pods within a large cluster.

This label-selector mechanism ensures that the ReplicaSet precisely targets the intended pods and maintains the set number of replicas by replacing any failed pods. 

### ReplicaSet / ReplicationController Commands Table

| **Resource Type**    | **Use Case**                    | **Example Command**                                                   |
| -------------------- | ------------------------------- | --------------------------------------------------------------------- |
| Create Object        | Create from a definition file   | `kubectl create -f <filename>`                                        |
| View ReplicaSets/RC  | List replication controllers    | `kubectl get replicaset`  **or**  `kubectl get replicationcontroller` |
| Delete ReplicaSet/RC | Remove a replication controller | `kubectl delete replicaset <replicaset-name>`                         |
| Update Definition    | Replace object using YAML file  | `kubectl replace -f <filename>`                                       |
| Scale ReplicaSet/RC  | Change number of replicas       | `kubectl scale --replicas=<number> -f <filename>`                     |

## Deployments
Deployment is a kubernetes object. 
- A Deployment in Kubernetes is a higher-level controller that manages ReplicaSets, which in turn manage Pods.
- It enables declarative updates to applications — you describe your desired state and Kubernetes handles the rest (creation, scaling, rolling updates, rollback).

Example:
```yaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```
### Common kubectl Commands

| Task                     | Command                                                          |
| ------------------------ | ---------------------------------------------------------------- |
| Create deployment        | `kubectl apply -f deployment.yaml`                               |
| View deployments         | `kubectl get deployments`                                        |
| View pods                | `kubectl get pods`                                               |
| Scale deployment         | `kubectl scale deployment nginx-deployment --replicas=5`         |
| Update image             | `kubectl set image deployment/nginx-deployment nginx=nginx:1.26` |
| Rollback to previous rev | `kubectl rollout undo deployment nginx-deployment`               |
| Check rollout status     | `kubectl rollout status deployment nginx-deployment`             |
| Delete deployment        | `kubectl delete deployment nginx-deployment`                     |

## Namespaces
By default, when you create objects such as pods, deployments, and services in your cluster, they are placed within a default namespace. 
- The default namespace is automatically created during the Kubernetes cluster setup.

Additionally, several system namespaces are created at startup:

- kube-system: Contains core system components like network services and DNS, segregated from user operations to prevent accidental changes.
- kube-public: Intended for resources that need to be publicly accessible across users.

        A Namespace in Kubernetes is a virtual cluster within your physical cluster.It helps you organize resources and logically isolate workloads.
Example:
- Create NameSpace
```yaml
metadata:
  name: my-app
  namespace: dev
```
- Resource Quota + LimitRange (per namespace)
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    limits.memory: 8Gi
```
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limits
  namespace: dev
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 500m
    defaultRequest:
      memory: 256Mi
      cpu: 250m
    type: Container

```
### Why Use Namespaces?

| Purpose             | Description                                                      |
| ------------------- | ---------------------------------------------------------------- |
| **Isolation**       | Separate resources between teams, environments (dev, test, prod) |
| **Organization**    | Group related resources together                                 |
| **Access Control**  | Use RBAC to grant different permissions per namespace            |
| **Resource Quotas** | Limit CPU, memory, etc. usage by namespace                       |
 ### Common kubectl Commands

| Task                                | Command                                                |
| ----------------------------------- | ------------------------------------------------------ |
| List all namespaces                 | `kubectl get namespaces`                               |
| Create a namespace                  | `kubectl create namespace dev`                         |
| Delete a namespace                  | `kubectl delete namespace dev`                         |
| Use a namespace (temporarily)       | `kubectl get pods -n dev`                              |
| Set default namespace (for session) | `kubectl config set-context --current --namespace=dev` |

## Service
Kubernetes Services enables communication between various components within and outside of the application.

- A Service in Kubernetes is an abstraction that defines a logical set of Pods and a policy to access them, usually through a stable IP address and DNS name.
- It solves the problem of dynamic IPs for Pods by giving them a permanent way to communicate.

        Purpose: Expose a set of Pods

        Exposes via: IP address and DNS

        Types  : ClusterIP, NodePort, LoadBalancer, ExternalName

### Why Use a Service?
| Feature                | Description                                    |
| ---------------------- | ---------------------------------------------- |
| 🔁 **Stable Endpoint** | Pods change, service IP remains constant       |
| 📡 **Load Balancing**  | Spreads traffic across backend Pods            |
| 🔎 **Discovery**       | Other components can discover services by name |
| 🔐 **Decoupling**      | Consumers don’t need to know Pod IPs           |


Types of Kubernetes Services
Kubernetes supports several service types, each serving a unique purpose:

- NodePort: Maps a port on the node to a port on a Pod. (Where the service makes an internal port accessible on a port on the NODE.)

        NodePort Service Breakdown
        With a NodePort service, there are three key ports to consider:

        - Target Port: The port on the Pod where the application listens (e.g., 80).

        - Port: The virtual port on the service within the cluster.

        - NodePort: The external port on the Kubernetes node (by default in the range 30000–32767).

Example:
```yaml
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - nodePort: 80      # Node port
      port: 80          # Exposed service port
      targetPort: 8080  # Pod's container port
  
```
- ClusterIP: Creates a virtual IP for internal communication between services (e.g., connecting front-end to back-end servers).

        - pods receive dynamic IP addresses that can change when they are recreated, relying on these IPs for internal communication is impractical.

        - Each service in Kubernetes is automatically assigned an IP and DNS name within the cluster. This Cluster IP should be used by other pods when accessing the service, ensuring consistent and reliable connectivity.
Example
```yaml
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80          # Exposed service port
      targetPort: 8080  # Pod's container port
  
```
- LoadBalancer: Provisions an external load balancer (supported in cloud environments) to distribute traffic across multiple Pods.

Example:
```yaml
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - nodePort: 80      # Node port
      port: 80          # Exposed service port
      targetPort: 8080  # Pod's container port
```
### ClusterIP vs NodePort vs LoadBalancer
| Feature           | ClusterIP              | NodePort         | LoadBalancer     |
| ----------------- | ---------------------- | ---------------- | ---------------- |
| In-cluster access | ✅                      | ✅                | ✅                |
| External access   | ❌                      | ✅ (nodeIP\:port) | ✅ (public IP)    |
| Use case          | Internal microservices | Dev/testing      | Production/cloud |

### Common kubectl Commands
| Task                          | Command                                       |
| ----------------------------- | --------------------------------------------- |
| View services                 | `kubectl get svc`                             |
| Describe service              | `kubectl describe svc my-service`             |
| Create from YAML              | `kubectl apply -f service.yaml`               |
| Delete service                | `kubectl delete svc my-service`               |
| Port forwarding (for testing) | `kubectl port-forward svc/my-service 8080:80` |

##  Imperative and Declarative
 Two core approaches are used to manage Kubernetes objects: imperative and declarative
 - Imperative Approach: You directly tell Kubernetes what to do using commands.
 - Declarative Approach: You declare the desired state of a resource in a YAML/JSON file, and Kubernetes ensures the current state matches it.


 | Feature         | Imperative                               | Declarative                          |
| --------------- | ---------------------------------------- | ------------------------------------ |
| Method          | CLI command (`kubectl create/run/scale`) | YSAML/JSON (`kubectl apply -f file`)  |
| Desired State   | Not stored                               | Defined in files                     |
| Use Case        | Quick tasks, one-offs                    | Production-grade, repeatable configs |
| Version Control | ❌ Hard                                   | ✅ Easy (files in Git)                |
| Idempotent      | ❌ No                                     | ✅ Yes                                |
