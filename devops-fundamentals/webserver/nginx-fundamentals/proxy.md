
## What is a Proxy?

A **proxy** is an intermediate server that sits between a client and a destination server, handling requests and responses.

---

## Reverse Proxy vs Forward Proxy

| Feature               | **Forward Proxy**                            | **Reverse Proxy**                           |
| --------------------- | -------------------------------------------- | ------------------------------------------- |
| **Used by**           | The **client (user)**                        | The **server (service provider)**           |
| **Client knows?**     | Client *chooses* to use the proxy            | Client usually doesn't know it's used       |
| **Acts on behalf of** | Client                                       | Server                                      |
| **Typical Use Case**  | Access control, anonymity, caching for users | Load balancing, SSL offloading, API gateway |
| **Hides**             | The **client’s identity**                    | The **server’s identity**                   |
| **Example Use**       | Company firewall, internet censorship bypass | NGINX, Cloudflare, AWS ALB, K8s ingress     |

---

## Diagram Comparison

### Forward Proxy:

```
[Client] ---> [Forward Proxy] ---> [Internet/Server]
              (client chosen)
```

Used to:

* Bypass geo-restrictions
* Filter content
* Mask client IP

---

### Reverse Proxy:

```
[Client] ---> [Reverse Proxy] ---> [Internal Servers]
              (server side)
```

Used to:

* Distribute load to multiple backend servers
* Add security (hide internal network)
* Handle SSL (TLS termination)
* Cache static content

---

## 🌐 Real-World Examples

| Use Case                    | Proxy Type    | Example Tools                 |
| --------------------------- | ------------- | ----------------------------- |
| Browsing blocked websites   | Forward Proxy | Squid Proxy, Shadowsocks      |
| Load balancing web traffic  | Reverse Proxy | NGINX, HAProxy, AWS ALB       |
| Hiding backend server IP    | Reverse Proxy | Cloudflare, Apache mod\_proxy |
| Enterprise internet gateway | Forward Proxy | Zscaler, Blue Coat            |


