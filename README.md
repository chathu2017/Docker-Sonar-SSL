# 🔐 Secure Jenkins & SonarQube Deployment using Docker, Nginx, and SSL (Let's Encrypt)

This guide provides a secure and cost-effective setup for deploying **Jenkins** and **SonarQube** using Docker and Docker Compose on an Ubuntu server. The services are securely exposed via **Nginx reverse proxy** with **SSL certificates from Let's Encrypt**, and access is limited to trusted IPs.

---

## ✅ Prerequisites

- **Server**: Ubuntu 22.04 with a valid public IP (`YOUR_SERVER_IP`)
- **Access**: Root or sudo privileges
- **DNS**:
  - `jenkins.examplesite.com` → `YOUR_SERVER_IP`
  - `sonar.examplesite.com` → `YOUR_SERVER_IP`
- **Tools**:
  - Docker and Docker Compose installed on the server

> ⚠️ DNS propagation might take a few minutes to hours.

---

## 🧠 Architecture & Strategy

1. **Firewall Setup (UFW)**: Deny access to ports 8080 & 9000, allow only SSH, HTTP, HTTPS.
2. **Containerized Services**: Jenkins and SonarQube run inside containers, exposed only to localhost.
3. **Nginx Reverse Proxy**: Routes traffic securely from subdomains to internal services.
4. **SSL with Certbot**: Free SSL certificates with auto-renewal.

---

## 🔒 Step 1: Secure Your Server (UFW)

```bash
# Allow SSH so you don't get locked out
sudo ufw allow ssh

# Allow HTTP & HTTPS for Nginx
sudo ufw allow 'Nginx Full'

# Block internal service ports from public access
sudo ufw deny 8080
sudo ufw deny 9000

# Enable the firewall
sudo ufw enable

# Check rules
sudo ufw status
