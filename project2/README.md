# 🖥️ The Server Commander — Project 2

> **Scenario:** A startup is launching a new dynamic application and needs a dedicated server environment with full OS control to install custom software and security patches.

---

## 📋 Overview

This project demonstrates how to act as a **SysAdmin** and provision a cloud-based virtual server from scratch — launching an EC2 instance, securing it via SSH, installing a web server, and hosting a custom webpage. No managed services, no GUIs — just the terminal.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Cloud Provider | AWS EC2 |
| OS | Ubuntu 22.04 LTS |
| Web Server | Nginx 1.24 |
| Access | SSH (key-pair authentication) |
| Security | AWS Security Groups + UFW Firewall |

---

## 🚀 Step-by-Step Setup

### Step 1 — Launch EC2 Instance (AWS Console)

1. Go to **AWS Console → EC2 → Launch Instance**
2. Configure:
   - **Name:** `decodelabs-server`
   - **AMI:** Ubuntu Server 22.04 LTS (Free Tier eligible)
   - **Instance Type:** `t2.micro` (Free Tier)
   - **Key Pair:** Create new → download `.pem` file
   - **Security Group:** Allow **SSH (22)**, **HTTP (80)**, **HTTPS (443)**
3. Click **Launch Instance**

---

### Step 2 — Connect via SSH

```bash
# Fix permissions on your key file (required on Linux/Mac)
chmod 400 your-key.pem

# Connect to your instance
ssh -i "your-key.pem" ubuntu@<YOUR_EC2_PUBLIC_IP>
```

> Replace `<YOUR_EC2_PUBLIC_IP>` with your instance's public IPv4 address from the EC2 dashboard.

---

### Step 3 — Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

---

### Step 4 — Install Nginx

```bash
# Install Nginx
sudo apt install nginx -y

# Start and enable on boot
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify it's running
sudo systemctl status nginx
```

Expected output:
```
● nginx.service - A high performance web server and a reverse proxy server
   Active: active (running) since ...
```

---

### Step 5 — Configure Firewall (UFW)

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

---

### Step 6 — Host the Custom Webpage

```bash
# Navigate to the web root
cd /var/www/html

# Remove default Nginx page
sudo rm index.nginx-debian.html

# Create your custom page
sudo nano index.html
```

Paste in the contents of `index.html` from this repository, then save (`Ctrl+O`, `Enter`, `Ctrl+X`).

```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/html

# Reload Nginx
sudo systemctl reload nginx
```

---

### Step 7 — Verify

Open your browser and navigate to:

```
http://<YOUR_EC2_PUBLIC_IP>
```

You should see the **Welcome to DecodeLabs** page live! ✅

---

## 📁 Project Structure

```
server-commander/
├── index.html          # Custom "Welcome to DecodeLabs" webpage
└── README.md           # This file
```

---

## 🔐 Security Notes

- SSH access is restricted to key-pair authentication only (password auth disabled)
- AWS Security Groups act as the first layer of defense
- UFW (Uncomplicated Firewall) provides OS-level traffic filtering
- Only ports 22, 80, and 443 are exposed to the internet

---

## 🧠 Key Concepts Practiced

- **Cloud provisioning** — spinning up and configuring a virtual machine on AWS
- **SSH security** — key-pair authentication and secure remote access
- **Linux CLI** — package management, file permissions, service management
- **Web server config** — installing, starting, and verifying Nginx
- **Firewall rules** — UFW + AWS Security Groups layered approach

---

## 📸 Result

The server is live and accessible via HTTP, serving a custom terminal-themed webpage that reflects the full provisioning workflow.

---

## 👤 Author

**Badar** — Software Engineering Student  
Project 2 | DecodeLabs | The Server Commander
