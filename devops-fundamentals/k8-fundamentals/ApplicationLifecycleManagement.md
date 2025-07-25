## Application Lifecycle Management

### What is Application Lifecycle Management?
Application Lifecycle Management refers to the entire process of managing an application in Kubernetes — from deployment to maintenance, updates, and eventual removal.

### Phases of Application Lifecycle
| Phase                       | Description                         | Tools/Commands                                               |
| --------------------------- | ----------------------------------- | ------------------------------------------------------------ |
| **1. Deployment**           | Launch app in the cluster           | `kubectl apply`, `Deployment`, `StatefulSet`, `helm install` |
| **2. Update/Rollout**       | Rolling out new versions or updates | `kubectl rollout`, image updates, `helm upgrade`             |
| **3. Scaling**              | Adjust replicas for load            | `kubectl scale`, `HorizontalPodAutoscaler`                   |
| **4. Monitoring & Logging** | Observe app behavior                | `kubectl logs`, Prometheus, Grafana, ELK                     |
| **5. Troubleshooting**      | Diagnose issues                     | `kubectl describe`, `kubectl get events`, probes             |
| **6. Rollback**             | Revert to previous version          | `kubectl rollout undo`, `helm rollback`                      |
| **7. Deletion**             | Remove resources                    | `kubectl delete`, `helm uninstall`                           |

### Kubernetes Resources Used
| Resource      | Purpose                                           |
| ------------- | ------------------------------------------------- |
| `Deployment`  | Declarative updates and scaling of stateless apps |
| `StatefulSet` | Manages stateful apps (e.g., DBs)                 |
| `DaemonSet`   | Ensures a pod runs on all (or some) nodes         |
| `Job/CronJob` | One-time or scheduled tasks                       |
| `Service`     | Exposes app internally or externally              |
| `Ingress`     | Manages external HTTP/HTTPS access                |

## Update Strategies

| Strategy                      | Description                                      |
| ----------------------------- | ------------------------------------------------ |
| **Rolling Update**            | Gradual update without downtime (default)        |
| **Recreate**                  | Stops all pods before starting new ones          |
| **Blue/Green** (manual)       | Deploy new version alongside old, switch traffic |
| **Canary** (manual/automated) | Deploy to subset of users, then roll out fully   |

## Probes for Health Management
| Probe               | Purpose                                             |
| ------------------- | --------------------------------------------------- |
| **Liveness Probe**  | Detect if the container is stuck and restart it     |
| **Readiness Probe** | Indicates if the app is ready to serve traffic      |
| **Startup Probe**   | Helps slow-starting containers avoid false restarts |

## Kubernetes Update Strategies – Comparison Table
| Strategy                        | Description                                           | Behavior                                                             | Downtime                       | Use Case                                                                 | Example                                                              | Limitations                                                                                      |
| ------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------ | ------------------------------------------------------------------------ | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Rolling Update** *(default)*  | Gradually replaces old pods with new ones             | New pods are created before old ones are terminated                  | ❌ No (if configured correctly) | Zero-downtime updates for stateless apps                                 | `strategy.type: RollingUpdate`<br>`maxSurge: 1`, `maxUnavailable: 1` | - May cause issues if app needs full restart<br>- Doesn't support database schema changes easily |
| **Recreate**                    | Deletes all old pods, then creates new ones           | All old pods removed before creating new ones                        | ✅ Yes (brief)                  | When new and old versions cannot co-exist (e.g., schema incompatibility) | `strategy.type: Recreate`                                            | - Causes complete downtime<br>- No availability during update                                    |
| **Blue/Green** *(manual)*       | Deploy new version separately and switch traffic      | Run both versions side by side, shift traffic manually or with tools | ❌ No (if switch is smooth)     | Safer rollout with rollback by switching service                         | Two deployments (blue & green) + Service switch                      | - Requires extra resources<br>- Needs manual traffic management or external tools                |
| **Canary** *(manual/automated)* | Send traffic to small % of new version, then scale up | Gradual exposure, allows testing before full rollout                 | ❌ No (if done gradually)       | Testing features with real users without full exposure                   | Use labels, selectors, or ingress weight routing                     | - Complex to manage manually<br>- Needs traffic routing logic or service mesh like Istio/Linkerd |

## Kubernetes Rollout & Update Command Table
| Command                                                         | Description                                               | Example Output                               |
| --------------------------------------------------------------- | --------------------------------------------------------- | -------------------------------------------- |
| `kubectl rollout status deployment <name>`                      | Check the status of the current rollout                   | Shows progress of pod updates                |
| `kubectl rollout history deployment <name>`                     | View deployment revision history                          | Lists revision numbers and change reasons    |
| `kubectl rollout undo deployment <name>`                        | Roll back to the previous deployment revision             | Restores the last working version            |
| `kubectl rollout undo deployment <name> --to-revision=<num>`    | Roll back to a specific revision                          | Useful when multiple rollbacks have occurred |
| `kubectl describe deployment <name>`                            | View detailed deployment info, including rollout strategy | Shows update strategy, events, etc.          |
| `kubectl edit deployment <name>`                                | Opens deployment manifest for inline editing              | Modify strategy, replicas, image, etc.       |
| `kubectl apply -f <file>.yaml`                                  | Apply/update deployment definition from YAML file         | Triggers rolling update                      |
| `kubectl set image deployment/<name> <container>=<image>:<tag>` | Trigger an update by changing the container image         | Rolls out new image version                  |
| `kubectl scale deployment <name> --replicas=<n>`                | Manually scale the number of pod replicas                 | Adjusts deployment size                      |
| `kubectl delete deployment <name>`                              | Deletes the deployment entirely                           | All pods are terminated                      |

## Environment Variables in Kubernetes
Environment variables in Kubernetes are used to pass configuration data to containers at runtime.

### Setting Environment Variables Using Docker
-  you can set environment variables using the -e flag. 
```
docker run -e APP_COLOR=pink simple-webapp-color
```

### Configuring Environment Variables in Kubernetes Pods
- Kubernetes allows you to define environment variables within your pod definitions in `env` block.
-  Each entry in the array should specify: name and value 
```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

### Leveraging ConfigMaps and Secrets for Environment Variables

- To define an environment variable directly, use:
```yml
env:
  - name: APP_COLOR
    value: pink
```
- To reference a ConfigMap for the environment variable, update the definition as follows:
```yml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: color
```
- Similarly, to source the environment variable from a Secret, configure it like this:
```yml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: color
```

Note: 
- Using ConfigMaps and Secrets promotes better security practices and easier management of configuration drift. Ensure these objects are updated consistently with your application requirements.

- Avoid hardcoding sensitive data directly into your manifests. Always use Secrets when dealing with sensitive information such as passwords or API keys.


### ConfigMaps & Configure ConfigMaps in Applications

#### What is a ConfigMap?
- A ConfigMap is a Kubernetes object used to store non-sensitive configuration data in key-value pairs.

- Use it to decouple config from container image and make applications more flexible and portable.

#### Ways to Create a ConfigMap
| Method        | Command / Syntax | Example                                                                  |
| ------------- | ---------------- | ------------------------------------------------------------------------ |
| From literal  | CLI              |`kubectl create configmap <configmap-name> --from-literal=<key>=<value>` <br>  or <br>  `kubectl create configmap app-config --from-literal=APP_MODE=production` |
| From file     | CLI              | `kubectl create configmap app-config --from-file=config.txt`             |
| From env file | CLI              | `kubectl create configmap app-config --from-env-file=.env`               |
| From YAML     | Declarative      | See below                                                                |

#### YAML Example to Define a ConfigMap
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  LOG_LEVEL: "debug"
  PORT: "8080"

```
#### Configure ConfigMaps in Applications

- As Environment Variables

Use when your app reads config from ENV vars.
```yml
env:
  - name: APP_MODE
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_MODE
```
- Using envFrom (All keys as ENV vars)

 Automatically inject all keys from the ConfigMap as environment variables.
```yml
envFrom:
  - configMapRef:
      name: app-config  
```

- Mounted as Files in Volume

Keys become filenames under /etc/config, values become file contents.
```yml
volumes:
  - name: config-volume
    configMap:
      name: app-config
containers:
  - name: myapp
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
```

## Secrets in Kubernetes
### What is a Secret?
A Kubernetes Secret is an object used to store sensitive data such as:

- Passwords

- API keys

- SSH tokens

- TLS certificates

Unlike ConfigMaps, Secrets are base64 encoded and are intended to protect confidential information.

### Types of Secrets

### Ways to Create a Secret
| Method        | Command                         | Example                                                                    |
| ------------- | ------------------------------- | -------------------------------------------------------------------------- |
| From literals | `kubectl create secret generic` | `kubectl create secret generic db-secret --from-literal=password=admin123` |
| From file     | `--from-file`                   | `kubectl create secret generic cert-secret --from-file=cert.pem`           |
| From YAML     | Declarative method              | See below                                                                  |
| From env file | `--from-env-file`               | `kubectl create secret generic app-secret --from-env-file=secret.env`      |

#### Example YAML (Opaque Secret)
```yml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=        # base64 for "admin"
  password: YWRtaW4xMjM=     # base64 for "admin123"
```
 You can encode using:
 ```
 echo -n "admin123" | base64
```

### How to Use Secrets in Pods
- As Environment Variables
```yml
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: username
```
- As Volume Mounts (Files)
```yml
volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
containers:
  - name: myapp
    volumeMounts:
      - name: secret-volume
        mountPath: /etc/secret
```
- Using envFrom to Load All Keys
```yml
envFrom:
  - secretRef:
      name: db-secret
```

### Limitations
| Limitation                       | Notes                                             |
| -------------------------------- | ------------------------------------------------- |
| Base64 ≠ encryption              | It’s just encoding; not secure by itself          |
| Not auto-updated in running pods | Pods must be restarted or watched manually        |
| Stored in etcd                   | Must secure etcd with encryption and restricted access |

Here are some key considerations:

- Secrets offer only Base64 encoding. For enhanced security, consider enabling encryption at rest for etcd.
- Limit access to Secrets using Role-Based Access Control (RBAC). Restrict permissions to only those who require it.
- Avoid storing sensitive secret definition files in source control systems that are publicly accessible.
- For even greater security, explore third-party secret management solutions such as AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or Vault.

*Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:*

- Not checking in secret object definition files to source code repositories.
- Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD.

Also, the way Kubernetes handles secrets. Such as:

- A secret is only sent to a node if a pod on that node requires it.

- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.

- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.



Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, and HashiCorp Vault.

## Encrypting Secret Data at Rest – Kubernetes
Reference: https://notes.kodekloud.com/docs/CKA-Certification-Course-Certified-Kubernetes-Administrator/Application-Lifecycle-Management/Demo-Encrypting-Secret-Data-at-Rest

Kubernetes uses an encryption provider configuration to determine how data is encrypted before storing it in etcd.

The kube-apiserver handles encryption transparently:

- When reading → Decrypts

- When writing → Encrypts (based on provider)
### Encryption Providers Supported
| Provider      | Description                                                             |
| ------------- | ----------------------------------------------------------------------- |
| **identity**  | No encryption (plaintext)                                               |
| **aesgcm**    | AES-GCM mode encryption (strongest, recommended)                        |
| **aescbc**    | AES-CBC mode with HMAC                                                  |
| **secretbox** | Based on NaCl (sodium) library                                          |
| **kms**       | External Key Management Service (e.g., AWS KMS, Azure Key Vault, Vault) |





### 🔐 **Encrypting Secret Data at Rest – How It Works (Step-by-Step)**

---

#### **1. Admin Enables Encryption at Rest**

1. **Generate an encryption key** (base64-encoded 32-byte key):

   ```bash
   head -c 32 /dev/urandom | base64
   ```

2. **Create `EncryptionConfiguration` YAML**:

   ```yaml
   apiVersion: apiserver.config.k8s.io/v1
   kind: EncryptionConfiguration
   resources:
     - resources:
         - secrets
       providers:
         - aescbc:
             keys:
               - name: key1
                 secret: <base64-key>
         - identity: {}  # fallback for decryption
   ```

3. **Configure kube-apiserver** to use the file:

   ```bash
   --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
   ```

4. **Restart kube-apiserver** for the config to take effect.

---

#### **2. API Server Encrypts New Secrets**

**When you create a Secret:**

```bash
kubectl create secret generic mysecret --from-literal=key=val
```

🔐 Behind the scenes:

* The **kube-apiserver** receives the request.
* It **checks the `EncryptionConfiguration`** file.
* If the resource type (`secrets`) is listed:

  * It uses the **top-most provider** (e.g. `aescbc`) to encrypt.
* Encrypted secret is written to **etcd**.

✅ Result: Even if someone accesses etcd storage, they’ll only see encrypted data.

---

#### **3. API Server Decrypts on Read**

When you retrieve a secret:

```bash
kubectl get secret mysecret -o yaml
```

🔍 What happens:

* The **kube-apiserver** reads the encrypted secret from etcd.
* It tries **each decryption provider** in the order listed.
* Once decrypted, it sends the **plaintext** result back to the user/pod.

---

#### **4. Key Rotation (Optional)**

To rotate the encryption key:

1. Add a new key at the top of the `keys:` list.
2. Restart kube-apiserver.
3. Re-encrypt existing secrets:

   ```bash
   kubectl get secrets --all-namespaces -o yaml | kubectl replace -f -
   ```

This forces re-encryption using the **new top key**.

---

#### 5. (Optional) Use External KMS**

For better security and key lifecycle management:

* Use the **KMS provider** with AWS KMS, Azure Vault, GCP KMS, or HashiCorp Vault.
* The kube-apiserver contacts the KMS plugin to encrypt/decrypt instead of doing it locally.

---

## 🧩 In Summary: How Encryption at Rest Works

| Stage            | Action                                                    |
| ---------------- | --------------------------------------------------------- |
| **Write Secret** | Kube-apiserver encrypts using `EncryptionConfiguration`   |
| **Store Secret** | Encrypted data is saved in etcd                           |
| **Read Secret**  | Kube-apiserver decrypts and returns plaintext             |
| **Rotate Key**   | Update config → restart API server → re-encrypt resources |




## Comparison of Secret Store CSI Driver vs External Secrets Operator (ESO) vs Sealed Secrets
Reference: 
- Secret Store CSI Driver : https://www.youtube.com/watch?v=MTnQW9MxnRI
- External Secrets Operator: https://www.youtube.com/watch?v=EonWeoFPpvM&t=1309s
- Sealed Secrets: https://www.youtube.com/watch?v=wWMJCY2E0d4
Here’s a clear breakdown of **how Secret Store CSI Driver, External Secrets Operator (ESO), and Sealed Secrets work** — including flow diagrams (text-based), key components, and step-by-step logic for each.



### 🔐 1. Secret Store CSI Driver (SSCD)

#### **How It Works**

```
[External Secret Store]
       |
       v
[SecretProviderClass YAML] ---> [CSI Driver + Cloud Provider Plugin]
                                        |
                                        v
                     [Mount secret as a file inside Pod at /mnt/secrets-store]
```

#### **Key Components**

* `SecretProviderClass`: Defines external secrets to fetch
* CSI Volume: Mounts secrets directly into pod
* Provider plugin (AWS, Azure, etc.)

#### **Step-by-Step**

1. Admin installs CSI driver and cloud provider plugin.
2. Create a `SecretProviderClass` to specify secrets to fetch.
3. In Pod spec, define a volume using CSI and reference the class.
4. CSI driver fetches the secret at **pod start**, mounts it as a file.
5. Application reads the secret from `/mnt/secrets-store`.

#### Best For

* Secrets that must **not be stored in etcd**
* Apps that read secrets as **files**

---

### 2. External Secrets Operator (ESO)

#### **How It Works**

```
[External Secret Store]
       |
       v
[ExternalSecret YAML]
       |
       v
[ESO Controller] ---> [Creates native Kubernetes Secret]
                               |
                               v
                    [Pod consumes it via env/volume]
```

#### **Key Components**

* `ExternalSecret`: Declares what to fetch and from where
* ESO Controller: Syncs secrets from store → `Secret`
* Kubernetes `Secret`: Standard K8s object, now populated

#### **Step-by-Step**

1. Admin installs ESO controller and provider configuration.
2. Create an `ExternalSecret` that links to external store and key.
3. ESO periodically fetches the secret and creates a Kubernetes `Secret`.
4. Apps use this native secret as env vars or mount as file.
5. Secrets are auto-rotated if external value changes.

#### Best For

* **Automatic syncing** from cloud secret managers
* Keeping secrets **as Kubernetes Secrets** (for compatibility)

---

### 3. Sealed Secrets (Bitnami)

#### **How It Works**

```
[User runs kubeseal CLI]
       |
       v
[SealedSecret YAML (encrypted)] ---> [Git Repo]
                                             |
                                             v
                                [Sealed Secrets Controller]
                                             |
                                             v
                            [Decrypts into Kubernetes Secret at runtime]
```

#### **Key Components**

* `kubeseal` CLI: Encrypts secrets into `SealedSecret` CRD
* `SealedSecret`: Custom resource, Git-safe
* Sealed Secrets Controller: Decrypts and creates K8s `Secret`

#### **Step-by-Step**

1. Create a regular `Secret` (locally).
2. Run `kubeseal` to encrypt it into a `SealedSecret` object.
3. Commit this encrypted file to Git.
4. Apply `SealedSecret` to cluster.
5. Controller decrypts it and creates a Kubernetes `Secret`.

#### Best For

* **GitOps workflows**
* Teams that **store everything in Git**, including secrets (securely)

---

### Summary Flow Comparison

| Step                         | CSI Driver                        | ESO                                | Sealed Secrets                           |
| ---------------------------- | --------------------------------- | ---------------------------------- | ---------------------------------------- |
| 1. Define source             | `SecretProviderClass`             | `ExternalSecret`                   | `kubeseal` CLI                           |
| 2. Where secrets are stored  | Cloud-only or optional K8s Secret | K8s Secret                         | Encrypted YAML in Git                    |
| 3. Secret access point       | Mounted file in pod               | K8s Secret (env/volume)            | K8s Secret (env/volume)                  |
| 4. Controller responsibility | Mount secret during pod creation  | Periodic sync from external source | Decrypt sealed secret into usable secret |
| 5. Rotation                  | Pod restart or manual             | Automatic                          | Manual reseal                            |




### Quick Summary

| Tool                                   | Purpose                                                                 |
| -------------------------------------- | ----------------------------------------------------------------------- |
| 🔐 **Secret Store CSI Driver (SSCD)**  | Mount secrets from external secret managers directly into pods as files |
| 🔁 **External Secrets Operator (ESO)** | Sync secrets from external secret stores into Kubernetes Secrets        |
| 🔒 **Sealed Secrets (Bitnami)**        | Encrypt secrets for safe storage in Git (GitOps)                        |

### Detailed Comparison Table
| Feature                       | **Secret Store CSI Driver**                                 | **External Secrets Operator (ESO)**                                                    | **Sealed Secrets**                                                          |
| ----------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| 🔧 **How it works**           | Mounts secrets directly into pods as files using CSI volume | Syncs secrets from external stores (AWS, Vault, etc.) into native K8s `Secret` objects | Encrypts `Secret` into `SealedSecret`, which can be safely committed to Git |
| 📂 **Secret format**          | File mount (`/mnt/secrets-store/secret`)                    | Kubernetes `Secret` object                                                             | Encrypted `SealedSecret` CRD                                                |
| 📦 **Secret stored in etcd**  | ❌ (unless explicitly synced)                                | ✅                                                                                      | ✅ (but encrypted)                                                           |
| 🌐 **External Store Support** | ✅ AWS, Azure, GCP, Vault, etc.                              | ✅ AWS, Azure, GCP, Vault, others                                                       | ❌ (works only with Kubernetes native secrets)                               |
| 🔐 **Security focus**         | Runtime-only access; no persistence                         | Secret centralization and syncing                                                      | GitOps + encryption                                                         |
| 🔁 **Secret rotation**        | Needs pod restart or sync tooling                           | ✅ Supports external rotation                                                           | Manual resealing required                                                   |
| 🔄 **Sync to K8s Secret**     | Optional                                                    | Default behavior                                                                       | Secrets stored as encrypted YAML                                            |
| ⚙️ **Workload access**        | Mount in pod as file                                        | Access like any Kubernetes Secret                                                      | Decrypt automatically in cluster                                            |
| 🔄 **GitOps friendly**        | ❌ (not Git-based)                                           | ⚠️ Somewhat, but secrets not in Git                                                    | ✅ Fully GitOps compatible                                                   |
| 🔑 **RBAC needed**            | App needs IAM or provider credentials                       | ESO controller needs secret access                                                     | Uses Kubernetes controller to decrypt                                       |


##  Multi-Container Pods in Kubernetes
A multi-container pod is a single Pod that runs more than one container.
These containers:

- Share the same network namespace

- Can communicate over localhost

- Can share volumes (storage)

✅ Use when containers need to work closely together (tight coupling).

| **Type**                         | **Runs When**            | **Primary Role**                                            | **Stops/Restarts**                 |
| -------------------------------- | ------------------------ | ----------------------------------------------------------- | ---------------------------------- |
| **Init Container**               | **Before main app**      | Perform setup tasks (e.g., Wait for database or external service) | - Runs once → then exits  <br> - Performs one-time setup before the main containers run.           |
| **Sidecar Container**            | **Alongside app**        | Provide supporting features (e.g., logging, proxy, metrics) | - Runs alongside main container <br> - Start before main container, stop after main container.      |
| **Main (Coordinated) Container** | Primary focus of the Pod | Runs the actual application logic                           | - Runs as long as the Pod is running <br> - No execution order	|

Example:

✅ Init container – waits for a fake database service

✅ Main (Coordinated) container – runs a basic NGINX web server

✅ Sidecar container – simulates a log forwarder (e.g., Fluentd)

```yml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-demo
  labels:
    app: multi-demo
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}

  # 🧱 Init container (runs first)
  initContainers:
    - name: init-wait-for-db
      image: busybox
      command: ['sh', '-c', 'echo "Waiting for DB..." && sleep 5']

  # 🚀 Main & Sidecar containers
  containers:
    # ✅ Main container (no order for below 2 main container)
    - name: nginx-main
      image: nginx
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
      ports:
        - containerPort: 80
      restartPolicy: Always
    - name: example
      image: nginx
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
      ports:
        - containerPort: 80
      restartPolicy: Always

    # ✅ Sidecar container
    - name: log-sidecar
      image: busybox
      command: ['sh', '-c', 'tail -n+1 -f /sidecar-logs/access.log']
      volumeMounts:
        - name: shared-logs
          mountPath: /sidecar-logs
      restartPolicy: Always
```
## Use restartPolicy: Always
Key Behavior
- If any container in the Pod fails, the entire Pod is restarted (unless it’s an Init container).

- Works well with controllers like Deployments that maintain desired state. 

For Pods created by Deployments, ReplicaSets, StatefulSets, DaemonSets, etc., restartPolicy: Always is set by default and cannot be changed.

### restartPolicy Options
| Policy      | Behavior                                                           |
| ----------- | ------------------------------------------------------------------ |
| `Always`    | Restart Pod if any container fails (default for Deployments, etc.) |
| `OnFailure` | Restart Pod **only if container exits with non-zero status**       |
| `Never`     | Never restart containers (even on failure)                         |


## When to Use restartPolicy: Always
| Scenario                           | Why It’s Used                                                |
| ---------------------------------- | ------------------------------------------------------------ |
| **Standard application workloads** | You want the app to restart automatically if it crashes      |
| **Multi-container pods**           | All containers (main + sidecars) should restart with the Pod |
| **Long-running services**          | Web servers, APIs, agents, loggers, etc.           

### When NOT to Use Always
If you're creating manual standalone pods (not managed by controllers) for tasks like:
| Use Case                             | Recommended `restartPolicy` |
| ------------------------------------ | --------------------------- |
| Batch jobs / once-only tasks         | `OnFailure` or `Never`      |
| Init setups (outside initContainers) | `OnFailure`                 |


## AutoScaling
Scaling refers to the process of adjusting the resources allocated to an application or system based on demand(or traffic).

In Kubernetes, this typically means:

- Increasing or decreasing the number of Pod replicas (horizontal)

- Increasing or decreasing the resources (CPU/Memory) assigned to Pods (vertical)

- Adjusting the number of Nodes in the cluster (cluster scaling)

### Types of Scaling
| **Type**                  | **What It Adjusts**            | **Example**                                   |
| ------------------------- | ------------------------------ | --------------------------------------------- |
| 🔁 **Horizontal Scaling** | Number of Pod replicas         | Scale from 3 to 10 Pods to handle more users  |
| 🧠 **Vertical Scaling**   | CPU/Memory of each Pod         | Change from 200Mi → 500Mi RAM for a heavy app |
| 🏗 **Cluster Scaling**    | Number of Nodes in the cluster | Add more EC2/GCE VMs when Pods can't fit      |

### Why Scaling Matters
| Benefit                | Explanation                                      |
| ---------------------- | ------------------------------------------------ |
| ⚡ **Performance**      | Handle traffic spikes without crashing           |
| 💸 **Cost-efficiency** | Avoid over-provisioning idle resources           |
| 🛠 **Resilience**      | Replace failed Pods automatically (self-healing) |
| 📏 **Flexibility**     | Adapt to unpredictable or seasonal workloads     |

### Manual vs. Auto Scaling
| Scaling Type       | Triggered By                 | Example                        |
| ------------------ | ---------------------------- | ------------------------------ |
| **Manual Scaling** | Human/user input             | `kubectl scale --replicas=5`   |
| **Auto Scaling**   | System metrics or thresholds | HPA scales Pods when CPU > 80% |

### HPA
HPA (Horizontal Pod Autoscaler) automatically scales the number of pod replicas in a Deployment, StatefulSet, or ReplicaSet based on observed metrics like CPU, memory, or custom metrics.

    🔄 It helps apps scale out under load and scale back down when idle — all automatically.

#### How It Works
- Monitors resource usage (via Metrics Server)

- Compares usage against the target utilization

- Calculates the required number of replicas

- Instructs the controller (e.g., Deployment) to scale accordingly

#### Prerequisites
- Metrics Server must be installed and running
```
kubectl top pod     # Should show CPU and memory usage
```
- Create HPA from CLI
```
kubectl autoscale deployment my-app \
  --cpu-percent=50 \
  --min=2 \
  --max=10
```
- YAML Example for HPA
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50

```                                               
|

### Limitations
| Limitation                 | Notes                                                 |
| -------------------------- | ----------------------------------------------------- |
| CPU/Memory only by default | Needs custom metrics adapter for other metrics        |
| Doesn’t work with Jobs     | Only works with long-running pods (e.g., Deployments) |
| Delay in scaling down      | HPA is reactive and needs time to stabilize           |
### Custom Metrics (Advanced)
You can use Prometheus or a custom adapter to scale based on:

- HTTP request count

- Queue length

- External metrics

## Vertical Pod Autoscaler (VPA)
VPA automatically adjusts the CPU and memory requests and limits of containers in your pods based on actual usage over time.

    🎯 It helps optimize resource allocation and reduce manual tuning of pod specs.

    VPA is not enabled by default, you must install it manually.

The VPA deployment includes three key components:

- **VPA Recommender:** Continuously monitors resource usage via the Kubernetes metrics API, analyzes historical and live data, and provides optimized recommendations for CPU and memory.
- **VPA Updater:** Compares current pod resource settings against recommendations and evicts pods running with suboptimal resources. This eviction triggers the creation of new pods with updated configurations.
- **VPA Admission Controller:** Intercepts pod creation requests and mutates the pod specification based on the recommender's suggestions, ensuring that new pods start with the ideal resource configuration.

### VPA vs HPA
| Feature        | **VPA**                          | **HPA**                       |
| -------------- | -------------------------------- | ----------------------------- |
| What it scales | **CPU/memory requests & limits** | **Number of pod replicas**    |
| Scope          | Individual **pod resources**     | Overall **workload scale**    |
| Pod restarts   | ✅ Yes (to apply changes)         | ❌ No (pods are added/removed) |
| Best for       | Memory/cpu tuning                | Handling load/throughput      |

#### Example VPA YAML
```yml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Options: "Off", "Initial", "Auto"
```

### updateMode Options
| Mode      | Behavior                                                |
| --------- | ------------------------------------------------------- |
| `Off`     | VPA only recommends; no automatic updates               |
| `Initial` | Recommendations only at pod creation                    |
| `Auto`    | VPA updates pods dynamically (deletes & recreates them) |

### Limitations
| Limitation                          | Notes                                        |
| ----------------------------------- | -------------------------------------------- |
| Restarts the pod to apply values    | Might interrupt service if not handled       |
| Not ideal with HPA on same resource | Use carefully or combine with care           |
| Not suitable for batch jobs         | Use only for long-running pods               |
| Can’t set **limits only**           | It modifies both **requests** and **limits** |
