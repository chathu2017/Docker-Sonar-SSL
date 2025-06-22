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
```

---

## 🐳 Step 2: Run Jenkins & SonarQube with Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts-jdk17
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - jenkins_data:/var/jenkins_home

  sonarqube:
    image: sonarqube:lts-community
    container_name: sonarqube
    restart: unless-stopped
    ports:
      - "127.0.0.1:9000:9000"
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"

volumes:
  jenkins_data:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
```

Then run:

```bash
docker-compose up -d
```

---

## 🌐 Step 3: Install Nginx and Certbot

```bash
# Update packages
sudo apt update

# Install Nginx & Certbot plugin
sudo apt install nginx python3-certbot-nginx -y
```

---

## 🔁 Step 4: Configure Nginx Reverse Proxy

### Jenkins

```bash
sudo nano /etc/nginx/sites-available/jenkins.examplesite.com
```

```nginx
server {
    listen 80;
    server_name jenkins.examplesite.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### SonarQube

```bash
sudo nano /etc/nginx/sites-available/sonar.examplesite.com
```

```nginx
server {
    listen 80;
    server_name sonar.examplesite.com;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable configs and reload:

```bash
sudo ln -s /etc/nginx/sites-available/jenkins.examplesite.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/sonar.examplesite.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔐 Step 5: Obtain SSL Certificates via Let's Encrypt

```bash
sudo certbot --nginx -d jenkins.examplesite.com -d sonar.examplesite.com
```

Follow the prompts to:
- Enter email
- Agree to terms
- Enable HTTPS redirection

---

## 🌍 Final Outcome

- **Jenkins**: https://jenkins.examplesite.com
- **SonarQube**: https://sonar.examplesite.com
- Services run securely via HTTPS.
- Ports 8080 & 9000 are completely private.

---

## ⏲️ Optional: Cron Job for Docker Restart Script

### Script Example

```bash
*/5 * * * * (echo "--- START CRON RUN: $(date) ---"; /home/exampleuser/docker-start-shell.sh; echo "--- END CRON RUN: $(date) ---") >> /home/exampleuser/cron-logs/docker_start.log 2>&1
```

### Explanation

- `*/5 * * * *` — Run every 5 minutes
- Logs grouped between `START` and `END` timestamps
- Errors are logged using `2>&1`
- Output saved to `/home/exampleuser/cron-logs/docker_start.log`

### How to Add

1. Open crontab:
   ```bash
   crontab -e
   ```

2. Add the new cron job line:
   ```bash
   */5 * * * * (echo "--- START CRON RUN: $(date) ---"; /home/exampleuser/docker-start-shell.sh; echo "--- END CRON RUN: $(date) ---") >> /home/exampleuser/cron-logs/docker_start.log 2>&1
   ```

3. Save and exit.

---

### ✅ Verify the Cron Job

Run this to see real-time logs:

```bash
tail -f /home/exampleuser/cron-logs/docker_start.log
```

Example output:

```log
--- START CRON RUN: Sat Jun 22 10:30:01 UTC 2025 ---
(Output from your script)
--- END CRON RUN: Sat Jun 22 10:30:02 UTC 2025 ---
```

> 🧹 Consider configuring logrotate to manage large log files.

---

## 👨‍💻 Author

This setup is ideal for startups, developers, or teams looking to bootstrap a secure CI/CD environment using Docker and open-source tools.

---

## 📜 License

MIT License. Use, modify, and distribute as needed.
