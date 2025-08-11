### Flow: How Kubernetes Handles a Request
- Authentication → Verify who you are.

- Authorization → Check if you’re allowed (RBAC/ABAC/Webhook).

- Admission → Validating/Mutating webhooks, policies (e.g., PodSecurityPolicy).

- Execution → API server processes the request

---
## Kubernetes Security Primitives
Kubernetes security is built on several **core primitives** that work together to protect the **control plane**, **nodes**, **network**, and **workloads**. These primitives help define **who can do what, where,** and **how** data and actions are secured.  

#### Securing Cluster Hosts
The security of your Kubernetes cluster begins with the hosts themselves. Protect your underlying infrastructure by following these practices:

- Disable root access.
- Turn off password-based authentication.
- Enforce SSH key-based authentication.
- Implement additional measures to secure your physical or virtual systems.

A compromised host can expose your entire cluster, so securing these systems is a critical first step.

### API Server Access Control
API Server Access Control
The Kube API server is at the heart of Kubernetes operations because all cluster interactions—whether via the kubectl command-line tool or directly through API calls—pass through it. Effective access control is essential, focusing on two key questions:

- Who can access the cluster?
- What actions are they permitted to perform?

### 1. Authentication  
Authentication verifies the identity of a user or service before granting access to the API server. Kubernetes offers various authentication mechanisms to suit different security needs:

- Static user IDs and passwords
- Tokens
- Client certificates
- Integration with external authentication providers (e.g., LDAP)

### 2. Authorization
After authentication, authorization determines what actions a user or service is allowed to perform. The **default mechanism, Role-Based Access Control (RBAC)**, associates identities with specific permissions. Kubernetes also supports:

- Attribute-Based Access Control (ABAC)
- Node Authorization
- Webhook-based authorization

These mechanisms enforce granular access control policies, ensuring that authenticated entities can perform only the operations they are permitted to execute.

### 3. Securing Component Communications
Secure communications between Kubernetes components are enabled via TLS encryption. This ensures that data transmitted between key components remains confidential and tamper-proof. Encryption protects:

- Communication within the etcd cluster
- Interactions between the Kube Controller Manager and Kube Scheduler
- Links between worker node components such as the Kubelet and Kube Proxy

### 4. Network Policies
By default, pods in a Kubernetes cluster communicate freely with one another. To restrict unwanted interactions and enhance security, Kubernetes provides network policies. These policies allow you to:

- Control traffic flow between specific pods
- Enforce security rules at the network level

## Authentication
Within a Kubernetes environment, there are different types of users:

- **Human Users:** Administrators and developers who perform administrative tasks and deploy applications.

- **Service Accounts:** Robot users that represent processes, services, or third-party applications.

        "Important Note"

        End-user application security is managed internally by the deployed applications. The focus here is solely on administrative access to the cluster.

#### Human User Authentication
- Kubernetes does not natively manage human user accounts. It **relies on external systems** for user identity verification.

- There are no built-in user objects in Kubernetes for user account.

Supported Authentication Methods

| Method                     | Description                                                                  | API Server Flag(s)                           |
| -------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------- |
| **Client Certificates**    | Auth via TLS using `x.509` certificates signed by the cluster CA.            | `--client-ca-file`                           |
| **Static Token File**      | CSV file with token, username, UID, groups.                                  | `--token-auth-file`                          |
| **Bootstrap Tokens**       | Temporary tokens for kubeadm/node join.                                      | Enabled internally                           |
| **Service Account Tokens** | Auto-mounted JWT tokens in pods. Verified using service account public keys. | `--service-account-key-file`                 |
| **OIDC (OpenID Connect)**  | External identity providers (Google, Okta, Azure, etc.).                     | `--oidc-*` flags                             |
| **Webhook Token Auth**     | Delegates authentication to external HTTP service.                           | `--authentication-token-webhook-config-file` |
| **Authentication Proxy**   | Relies on a trusted front-end proxy to inject user identity headers.         | `--requestheader-*` flags                    |


## TLS Certificates 


### TLS (Transport Layer Security)

**TLS** is a protocol that ensures **encryption**, **authentication**, and **data integrity** over a network. It’s the backbone of secure communication on the internet (HTTPS, for example).
Advance  Version of SSL.

### TLS Provides:

* **Confidentiality** (data is encrypted)
* **Integrity** (data isn’t altered)
* **Authentication** (identity of server, and optionally client, is verified)

---

## 📄 Certificates in TLS

A **certificate** is a digital file used to prove identity. It includes:

* Public key
* Information (domain, org, etc.)
* Issuer info
* Digital signature (from Certificate Authority - CA)

### 🔐 Private Key vs Public Key

* **Private Key**: Kept secret by owner (server or client)
* **Public Key**: Shared with everyone via certificate

---

## Server Certificate vs Client Certificate

| Type                   | Purpose                                | Common Use                     |
| ---------------------- | -------------------------------------- | ------------------------------ |
| **Server Certificate** | Authenticates the server to the client | HTTPS websites, APIs, etc.     |
| **Client Certificate** | Authenticates the client to the server | Mutual TLS (mTLS), secure APIs |

---

##  TLS Handshake (Simplified)

1. **Client** → Server: "Hello, I support these TLS versions + ciphers"
2. **Server** → Client: "Here's my certificate"
3. Client validates server certificate (using CA chain)
4. If valid, they generate shared encryption keys using asymmetric encryption
5. Optionally, **Client sends its own certificate** (if server asks for it)
6. Encrypted communication begins

---

## 🛠 How Certificates are Created

### 1. Generate Private Key

```bash
openssl genrsa -out ca.key 2048
```

### 2. Generate Certificate Signing Request (CSR)

```bash
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```
You’ll be asked:

* Country
* State
* Organization
* Common Name (e.g., domain.com)

### 3. Self Signed certifivate (optional)
```
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```




---

## 📂 File Summary

| File         | Description                       |
| ------------ | --------------------------------- |
| `server.key` | Private key                       |
| `server.crt` | Public certificate                |
| `server.csr` | Request to be signed by CA        |
| `ca.crt`     | CA public certificate             |
| `ca.key`     | CA private key (used for signing) |

---

## ✅ Verify Certificates

### Check certificate details:

```bash
openssl x509 -in server.crt -text -noout
```

### Check CSR:

```bash
openssl req -in server.csr -text -noout
```

### Verify that cert and key match:

```bash
openssl x509 -noout -modulus -in server.crt | openssl md5
openssl rsa -noout -modulus -in server.key | openssl md5
```

---

## Client Certificate Auth (Mutual TLS)

1. Server has its own cert/key
2. Server asks client to present cert
3. Client sends its own cert
4. Server validates client’s cert using CA

---

## 🔐 TLS in Real World (Example: NGINX)

```nginx
server {
    listen 443 ssl;
    ssl_certificate     /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;

    ssl_client_certificate /etc/nginx/certs/ca.crt;
    ssl_verify_client on;
}
```

---

## Summary

| Concept                    | Purpose                                   |
| -------------------------- | ----------------------------------------- |
| TLS                        | Secure communication                      |
| Certificate                | Proves identity with public key           |
| Private Key                | Used to decrypt and sign                  |
| Server Certificate         | Authenticates server to client            |
| Client Certificate         | Authenticates client to server (optional) |
| CA (Certificate Authority) | Signs and validates certs                 |





## What is `kubeconfig`?

* A configuration file used by `kubectl` (or other Kubernetes clients) to authenticate and interact with Kubernetes clusters.
* Default location: `~/.kube/config`



### 🔹 File Structure

A `kubeconfig` file contains:

* **clusters**: Kubernetes API server endpoints + TLS certs.
* **users**: Credentials to access the cluster.
* **contexts**: Links a user to a cluster.
* **current-context**: The context `kubectl` uses by default.

```yaml
apiVersion: v1
kind: Config
clusters:
- name: dev-cluster
  cluster:
    server: https://<cluster-endpoint>
    certificate-authority-data: <base64-cert>

users:
- name: dev-user
  user:
    token: <bearer-token>

contexts:
- name: dev-context
  context:
    cluster: dev-cluster
    user: dev-user

current-context: dev-context
```

---

## Common Commands

### View Config

```bash
kubectl config view
```

###  Set Context

```bash
kubectl config use-context <context-name>
```

### ➕ Add Cluster, User, Context

```bash
kubectl config set-cluster <name> --server=<URL> --certificate-authority=<path-to-ca>
kubectl config set-credentials <user> --token=<token>
kubectl config set-context <context> --cluster=<cluster> --user=<user>
```

### ❌ Delete Context/User/Cluster

```bash
kubectl config delete-context <context-name>
kubectl config delete-user <user-name>
kubectl config delete-cluster <cluster-name>
```

---

### Environment Variable Override

To use a different kubeconfig file:

```bash
export KUBECONFIG=/path/to/your/config
```

To merge multiple kubeconfigs:

```bash
export KUBECONFIG=config1:config2
kubectl config view --merge --flatten > merged_config
```


---

### ✅ **1. API Group**

Kubernetes organizes its APIs into **API groups** to manage versions and logical separation.
Organize based on specific functionality

* **Core group ("" - empty string)**

  Contains essential resources like: `Pod`, `Service`, `ConfigMap`, `Node`, `Secret`, `Namespaces`.

* **Named groups**

  Provides an organized structure for newer features. These groups include apps, extensions, networking, storage, authentication, and authorization. 
  Examples:

  * `apps`: Deployments, StatefulSets, DaemonSets
  * `batch`: Jobs, CronJobs
  * `rbac.authorization.k8s.io`: RBAC resources
  * `networking.k8s.io`: Ingress, NetworkPolicies

* **Format of API version:**
  `apiVersion: <group>/<version>`
  Example: `apps/v1`, `rbac.authorization.k8s.io/v1`

Note: To retrieve the list of available API groups, access the API server's root endpoint on port 6443:
```
curl http://localhost:6443 -k
```
---

### ✅ **2. Authorization in Kubernetes

#### What is Authorization?
Authorization checks “can this identity do this action on this resource?”

    Example:
    Can user dev-user create a pod in namespace dev?

#### Common Authorization Modes
Kubernetes supports multiple authorization modes, and they can be used together:


| Mode            | Description                                      |
| --------------- | ------------------------------------------------ |
| **RBAC**        | Role-Based Access Control (most widely used)     |
| **Node**        | Grants permissions to kubelets                   |
| **ABAC**        | Attribute-Based Access Control (deprecated/rare) |
| **Webhook**     | Sends auth requests to an external service       |
| **AlwaysAllow** | Approves all requests without performing any authorization checks (not for production)       |
| **AlwaysDeny**  | Denies all requests (rarely used)                |

---

### ✅ **3. RBAC (Role-Based Access Control)**

Controls *what* an authenticated user/service can do.

RBAC (Role-Based Access Control) is the **default and recommended** method.

It works by:

- Defining roles (what actions are allowed)

- Binding those roles to subjects (users, groups, service accounts)

RBAC is implemented using these resources:
| Resource             | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| `Role`               | Set of permissions in a namespace                   |
| `ClusterRole`        | Permissions cluster-wide                            |
| `RoleBinding`        | Assigns a Role to a subject in a namespace          |
| `ClusterRoleBinding` | Assigns a ClusterRole to subject across the cluster |


📌 **RBAC Structure**

```yaml
# Example: Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

```

```yaml
# Example: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]

# Example: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

---

### ✅ **4. ClusterRole vs Role**

| Feature                              | Role               | ClusterRole                       |
| ------------------------------------ | ------------------ | --------------------------------- |
| Scope                                | Namespace-specific | Cluster-wide or all namespaces    |
| Used with                            | RoleBinding        | ClusterRoleBinding or RoleBinding |
| Can access non-namespaced resources? | ❌                  | ✅                                 |

### Commands
How to Test Authorization
You can simulate and test if a request is allowed using:

```bash
kubectl get roles
kubectl get rolebindings
kubectl describe rolebinding devuser-developer-binding

kubectl auth can-i <verb> <resource> --namespace=<ns> --as=<user>

Example:
kubectl auth can-i get pods --as=dev-user --namespace=dev
kubectl auth can-i create deployment --as=system:serviceaccount:default:myapp-sa

```
Authorization Flow in Kubernetes
```
Request → Authentication → Authorization → Admission Control → Execution

```
---

### ✅ **5. ServiceAccount**

Used by pods to access the Kubernetes API.

* User accounts are meant for humans (admins/developers); Service Accounts (SAs) are used by applications, pods, and automation tools to interact with the Kubernetes API 

- When you create an SA (e.g. with kubectl create serviceaccount dashboard-sa), Kubernetes generates a secret token attached to that SA by default

* Each service account gets:

  * A **token** mounted in the pod (can be disabled).
  * Optionally, secrets, image pull credentials.
  * Can be assigned RBAC permissions via RoleBinding/ClusterRoleBinding.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
```

> 📌 Best Practice:
> Create dedicated service accounts for applications and restrict their access using RBAC.


## Kubernetes Image Security

Image Naming & Registry Awareness
- Image name with no registory path assume the official Docker Hub repository  by default. Example Unqualified image names (e.g. nginx) implicitly map to docker.io/library/nginx
- Always specify full registry paths for private or specific public registries (e.g., private-registry.io/myapp, gcr.io/k8s‑e2e/dnsutils)



### 🔸 1. **Use Trusted Registries**

* Pull container images **only** from trusted sources (e.g., Docker Hub, ECR, GCR, private registry).
* Avoid using `latest` tag—use pinned, versioned, immutable images.

```yaml
image: myregistry.io/myapp:v1.2.3   ✅
image: myapp:latest                 ❌
```

---

### 🔸 2. **ImagePullPolicy**

Controls when the image is pulled:

* `Always`: Always pull image (default if tag is `latest`)
* `IfNotPresent`: Pull if not already present
* `Never`: Use local image only

```yaml
imagePullPolicy: IfNotPresent
```

---

### 🔸 3. **Private Registry Access**

Use **Secrets** for registry authentication:

#### Create secret:

```bash
kubectl create secret docker-registry myregistrykey \
  --docker-server=REGISTRY_URL \
  --docker-username=USER \
  --docker-password=PASSWORD \
  --docker-email=EMAIL
```

#### Attach to service account or pod:

```yaml
spec:
  imagePullSecrets:
    - name: myregistrykey
```

---

### 🔸 4. **PodSecurityPolicy / SecurityContext (Deprecated → Use Pod Security Admission)**

Avoid running as root. Use `securityContext` to enforce:

```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
  readOnlyRootFilesystem: true
```

Also restrict capabilities:

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
```

---

### 🔸 5. **Image Scanning**

Scan images before deploying:

* Use tools like:

  * **Trivy** (Aqua Security)
  * **Clair**
  * **Grype**
  * **Twistlock**, **Anchore**, **Snyk**

Example:

```bash
trivy image myapp:v1.2.3
```

---

### 🔸 6. **Admission Controllers**

Use policies to enforce image security:

* **OPA/Gatekeeper**: Custom policies (e.g., block `latest`, require signed images)
* **Kyverno**: Kubernetes-native policy engine
* **Pod Security Admission (PSA)**: Enforces restricted modes

---

### 🔸 7. **Image Signing & Verification**

* Use **cosign**, **Notary v2**, or **Sigstore** to sign and verify images.
* Enforce signed images using OPA/Kyverno.

---

### 🔸 8. **Limit Pulling to Internal Networks**

Use network policies and firewall rules to:

* Block access to public registries (if not allowed)
* Ensure only internal nodes pull from private registries

---

## ✅ Best Practices Summary

| Practice                     | Purpose                       |
| ---------------------------- | ----------------------------- |
| Pin image tags               | Avoid unintentional updates   |
| Use private registries       | Control and audit images      |
| Avoid `latest` tag           | Prevent unstable builds       |
| Enable image scanning        | Detect CVEs                   |
| Enforce non-root containers  | Reduce attack surface         |
| Use `readOnlyRootFilesystem` | Prevent file system tampering |
| Enforce policies             | Ensure compliance             |
| Require image signing        | Verify authenticity           |

---

Summary:
- Always use explicit, versioned image tags and trusted registries.

- Use Kubernetes Docker registry secrets to authenticate access to private images.

- Add the secret correctly in pod/deployment specs via imagePullSecrets.

- Troubleshoot failures by checking secret correctness and registry access permissions.

---
##  Docker security

### Key Takeaways:

* **Process Isolation**: Docker uses Linux namespaces to isolate container processes from the host and from each other.
* **User Context Management**: By default, containers run as `root`, making processes inside containers appear as root on the host.
* **Run as Non-root**: Use the `--user` flag to run container processes under a specific, non-root user ID, enhancing security.
* **Managing Capabilities**: Docker allows granular control over Linux capabilities:

  * Use `--cap-add` to grant specific capabilities (e.g., `MAC_ADMIN`)
  * Use `--cap-drop` to remove unwanted capabilities
  * Avoid using `--privileged` in production—it grants broad, potentially dangerous privileges
  
  Here’s a polished summary of the **Security Contexts** topic from KodeKloud’s CKA course and related Kubernetes documentation:

---

## Security Contexts in Kubernetes (KodeKloud CKA Notes)

### Overview (KodeKloud):

* **Concept & Scope**
  Security contexts allow you to define security-related settings at both the **Pod level** and the **Container level** within Kubernetes. Settings in a container override those specified for the pod.([KodeKloud Notes][1])

* **Docker to Kubernetes Transition**
  In Docker, you can customize container security using commands like:

  ```bash
  docker run --user=1001 ubuntu sleep 3600
  docker run --cap-add MAC_ADMIN ubuntu
  ```

  Kubernetes extends this capability through declarative settings in Pod specs.([KodeKloud Notes][2])

* **Practical Example**
  A Pod spec may include a container-level security context, such as:

  ```yaml
  securityContext:
    runAsUser: 1000
    capabilities:
      add: ["MAC_ADMIN"]
  ```

  This ensures the container runs as user ID 1000 and is granted the `MAC_ADMIN` capability.([KodeKloud Notes][1])

---

## Kubernetes Best Practices (Broader K8s Guidance):

According to official Kubernetes documentation:

* **Security context** can define a wide range of settings, including:

  * Discretionary Access Control (UID/GID)
  * SELinux labels
  * Privileged mode toggles
  * Linux capabilities
  * AppArmor and Seccomp profiles
  * `allowPrivilegeEscalation` and `readOnlyRootFilesystem`([Kubernetes][1])

* **Hierarchy of Settings**: Pod-level settings like `runAsUser`, `runAsGroup`, `fsGroup`, and `supplementalGroups` apply to all containers, unless overridden at the container level.([Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/?utm_source=chatgpt.com))

---

## Combined Summary

| Level               | Applies To              | Common Settings                                                         |
| ------------------- | ----------------------- | ----------------------------------------------------------------------- |
| **Pod-level**       | All containers in a Pod | `runAsUser`, `runAsGroup`, `fsGroup`, `supplementalGroups`              |
| **Container-level** | Individual container    | `runAsUser`, capabilities, `privileged`, and more (overrides pod-level) |

### Key Takeaways:

* Apply security contexts at the **Pod level** for consistent settings, then **override at the container level** only if needed.
* Always run containers as **non-root users** (e.g., `runAsUser: 1000`).
* Restrict privileges tightly, adding only necessary capabilities and avoiding `privileged: true`.
* Utilize advanced security layers like SELinux, AppArmor, or Seccomp for enhanced protection.




---

## 🔐 NetworkPolicy

A `NetworkPolicy` is a Kubernetes resource that controls traffic **in and out of pods** based on **labels**, **namespaces**, and **IP blocks**. It works **only with supported CNI plugins** (like Calico, Cilium, etc.).

---

### **Key Concepts**

* **By default:** All pods **accept all traffic** from any source.
* Once a **NetworkPolicy** is applied to a pod (by label), that pod:

  * **Denies all ingress/egress** traffic **except** what’s explicitly allowed.
* NetworkPolicies:

  * Are **namespace-scoped**
  * Only apply to **Pod-to-Pod** and **external IP** traffic (not internal container traffic)

---

### 🛠️ **Basic Structure**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-some-traffic
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      role: db            # Target Pods
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend  # Allowed source Pods
    ports:
    - protocol: TCP
      port: 3306
```

---

### 🔄 **policyTypes**

* `Ingress`: Controls **incoming** traffic **to** pods
* `Egress`: Controls **outgoing** traffic **from** pods

If `policyTypes` is omitted, it is inferred from the presence of `ingress` or `egress` rules.

---

### 🧭 **Selectors**

* `podSelector`: Labels of the target pods **within the same namespace**
* `namespaceSelector`: Select namespaces (used with `podSelector` or alone)
* `ipBlock`: CIDRs and optional `except` list

**Example: allow from a specific IP range:**

```yaml
from:
- ipBlock:
    cidr: 10.0.0.0/16
    except:
    - 10.0.1.0/24
```

---

### ⚠️ **Important Behaviors**

* No **default deny** policy exists unless a `NetworkPolicy` is created.
* To implement **default deny**, create a policy with no `from` or `to`.

  ```yaml
  # Deny all ingress to all pods in namespace
  podSelector: {}  # all pods
  ingress: []
  ```

---

### 🧪 **Examples**

✅ **Allow Ingress to Pods with app=backend from Pods with app=frontend**

```yaml
podSelector:
  matchLabels:
    app: backend
ingress:
- from:
  - podSelector:
      matchLabels:
        app: frontend
```

✅ **Deny All Egress from a Pod**

```yaml
podSelector:
  matchLabels:
    app: isolated
policyTypes:
- Egress
egress: []
```

✅ **Allow DNS & HTTP Outbound (Egress)**

```yaml
egress:
- to:
  - namespaceSelector: {}
  ports:
  - port: 53
    protocol: UDP
  - port: 80
    protocol: TCP
```

---

### **Verify with:**

```bash
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <name> -n <namespace>
```

---

### Additional Tips

* Test policies using busybox or netshoot pods:

  ```bash
  kubectl run test --rm -it --image=busybox -- /bin/sh
  ```
* Use tools like **`kubectl exec`**, **`curl`**, **`nc`**, or **`ping`** to verify connectivity.

---




