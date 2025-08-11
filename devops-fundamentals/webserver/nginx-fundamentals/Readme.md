# NGINX Fundamentals

This repository provides a beginner-to-intermediate level guide on using **NGINX**, covering its key roles such as a web server, reverse proxy, load balancer, and SSL terminator.

## 📚 Topics Covered

| File                              | Description                                                   |
|-----------------------------------|---------------------------------------------------------------|
| `Introduction.md`                | Overview of NGINX and its architecture                        |
| `nginx-as-a-webserver.md`        | Setting up NGINX as a basic static file web server            |
| `nginx-as-a-reverse-proxy.md`    | Using NGINX to forward requests to backend servers            |
| `nginx-as-a-load-balancer.md`    | Load balancing strategies with NGINX                          |
| `nginx-with-ssl-or-tls.md`       | Configuring NGINX for SSL/TLS (HTTPS setup)                   |
| `proxy.md`                       | Difference between forward proxy and reverse proxy            |

## ✅ Requirements

- Basic knowledge of Linux and web architecture
- NGINX installed on your machine


## 🔧 Useful NGINX Commands

```bash
nginx -t         # Test configuration
nginx -s reload  # Reload NGINX with updated config
systemctl restart nginx   # Restart service (Linux)
