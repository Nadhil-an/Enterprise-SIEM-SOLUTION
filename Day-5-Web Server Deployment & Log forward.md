# Setup-05 — Web Server (Apache + DVWA) Deployment & Log Forwarding to Splunk

![AWS](https://img.shields.io/badge/Platform-AWS-FF9900?style=flat-square&logo=amazonaws)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%2022.04-E95420?style=flat-square&logo=ubuntu)
![Apache](https://img.shields.io/badge/Web%20Server-Apache2-D22128?style=flat-square&logo=apache)
![DVWA](https://img.shields.io/badge/App-DVWA-red?style=flat-square)
![Splunk](https://img.shields.io/badge/SIEM-Splunk-FF6B35?style=flat-square&logo=splunk)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

---

## 📋 Overview

An Ubuntu EC2 instance was deployed inside the SOC-VPC and configured as a vulnerable web server running Apache and DVWA (Damn Vulnerable Web Application). Apache access and error logs are forwarded to the Splunk SIEM via Universal Forwarder, enabling detection of web-based attacks including SQL injection, brute force, command injection, file upload, and local file inclusion. This server forms the attack surface for all 5 web attack detection labs.

---

## 🏗️ Architecture

```
Attacker Machine
      │
      ▼
Internet Gateway (SOC-IGW)
      │
      ▼
Ubuntu EC2 (Apache + DVWA)
      │
      ├── /var/log/apache2/access.log ──┐
      └── /var/log/apache2/error.log  ──┴──► Splunk Forwarder ──► Splunk SIEM (index=web_logs)
```

<div align="center">
<img src="assets/web-architecture.png" width="800">
</div>

---

## ⚙️ EC2 Instance Configuration

| Setting | Value |
|---|---|
| AMI | Ubuntu Server 22.04 LTS |
| Instance Type | t3.micro |
| Network | SOC-VPC |
| Subnet | SOC-Public-Subnet |
| Auto-assign Public IP | Enabled |
| Storage | 20GB gp2 |

---

## 🔐 Security Group — Inbound Rules

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH administration |
| 80 | TCP | HTTP — DVWA web access |
| 9997 | TCP | Splunk Universal Forwarder outbound |

---

## ⚙️ Deployment Steps

### Step 1 — Connect to EC2 via SSH

```bash
ssh -i your-key.pem ubuntu@<EC2-Public-IP>
```

---

### Step 2 — Install Apache Web Server

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

Verify Apache is running by visiting:

```
http://<EC2-Public-IP>
```


---

### Step 3 — Install PHP and MySQL

DVWA requires PHP and a MySQL database backend:

```bash
sudo apt install php php-mysqli php-gd libapache2-mod-php mysql-server git -y
sudo systemctl restart apache2
```

---

### Step 4 — Install and Configure DVWA

Clone DVWA into the Apache web root:

```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
sudo chown -R www-data:www-data DVWA
```

Create DVWA database:

```bash
sudo mysql
```

Inside MySQL:

```sql
CREATE DATABASE dvwa;
EXIT;
```

Configure DVWA database connection:

```bash
sudo cp /var/www/html/DVWA/config/config.inc.php.dist /var/www/html/DVWA/config/config.inc.php
```

Access DVWA setup page in browser:

```
http://<EC2-Public-IP>/DVWA/setup.php
```

Click **Create / Reset Database**, then login:

| Field | Value |
|---|---|
| Username | admin |
| Password | password |

Set security level to **Low** for attack simulation labs.


<div align="center">

<img src="assets/web-dvwa.png" width="800">

</div>

---

### Step 5 — Install Splunk Universal Forwarder

```bash
wget -O splunkforwarder.tgz "https://download.splunk.com/products/universalforwarder/releases/10.2.0/linux/splunkforwarder-10.2.0-linux-amd64.tgz"
tar -xvzf splunkforwarder.tgz
sudo mv splunkforwarder /opt/
cd /opt/splunkforwarder/bin
sudo ./splunk start --accept-license
sudo ./splunk enable boot-start
```

---

### Step 6 — Connect Forwarder to Splunk Server

```bash
sudo ./splunk add forward-server <Splunk-Server-IP>:9997
```

Verify connection:

```bash
sudo ./splunk list forward-server
```

---

### Step 7 — Forward Apache Logs to Splunk

Add Apache access log:

```bash
sudo ./splunk add monitor /var/log/apache2/access.log -index web_logs -sourcetype access_combined
```

Add Apache error log:

```bash
sudo ./splunk add monitor /var/log/apache2/error.log -index web_logs -sourcetype apache_error
```

Restart forwarder to apply:

```bash
sudo ./splunk restart
```

---

### Step 8 — Verify Logs in Splunk

On the Splunk server, confirm web logs are being received:

```spl
index=web_logs
| stats count by status, clientip
| sort - count
```

Expected fields in logs:

| Field | Description |
|---|---|
| clientip | Source IP of request |
| method | HTTP method (GET/POST) |
| uri | Requested endpoint |
| status | HTTP response code |
| useragent | Client browser/tool |


<div align="center">
<img src="assets/web-splunk.png" width="800">
</div>
---

## 📋 Log Configuration Summary

| Log File | Index | Sourcetype |
|---|---|---|
| `/var/log/apache2/access.log` | `web_logs` | `access_combined` |
| `/var/log/apache2/error.log` | `web_logs` | `apache_error` |

---

## 🔎 Detection Queries Used Across Web Labs

```spl
-- SQL Injection detection
index=web_logs | search "%27" | stats count by clientip

-- Brute force detection
index=web_logs | search "/brute/" | stats count by clientip | sort -count

-- Command injection detection
index=web_logs | search "/exec/" AND ("%3B" OR "whoami") | stats count by clientip, uri

-- Web shell detection
index=web_logs | search "shell.php" AND "cmd=" | stats count by clientip, uri

-- LFI / path traversal detection
index=web_logs | search "../" | stats count by clientip, uri
```

---

## ✅ Deployment Summary

| Component | Status |
|---|---|
| Ubuntu EC2 launched | ✅ |
| Apache2 installed and running | ✅ |
| PHP and MySQL installed | ✅ |
| DVWA deployed and accessible | ✅ |
| Splunk Universal Forwarder installed | ✅ |
| Apache logs forwarding to `index=web_logs` | ✅ |
| Logs verified in Splunk | ✅ |
---

*SOC Lab Infrastructure | Apache2 + DVWA | Ubuntu 22.04 | AWS Free Tier | Author: [Nadil](https://github.com/Nadhil-an)*
