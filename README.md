# ☁️ Cloud Computing 

> A hands-on series of cloud computing projects completed during my internship at **DecodeLabs**, covering server provisioning, static hosting, security, and global deployment on AWS and Azure.

---

## 📦 Projects at a Glance

| # | Project | Cloud Service | Hosting Model | Key Skill |
|---|---|---|---|---|
| 2 | [The Server Commander](#-project-2--the-server-commander) | AWS EC2 | Virtual Machine | SysAdmin, SSH, Nginx |
| 3 | [The Global Launch](#-project-3--the-global-launch) | AWS S3 / Azure Blob | Serverless Static | Object Storage, CDN |

---

## 🌍 Project 1 — The Global Launch

> **Scenario:** Deploy a personal portfolio as a static site on cloud object storage — no servers, no provisioning, instant global access.

### Overview

This project demonstrates hosting a **static portfolio website** using cloud object storage. The site loads instantly worldwide, costs near zero, and scales infinitely — without provisioning a single server.

### Tech Stack

| Layer | Technology |
|---|---|
| Cloud Provider | AWS S3 or Azure Blob Storage |
| CDN (optional) | AWS CloudFront / Azure CDN |
| Hosting Model | Serverless static website |
| Access Control | Bucket Policy / Public Read |

### Deployment Guide

#### Option A — AWS S3

```bash
# 1. Create bucket
aws s3 mb s3://your-portfolio-name --region us-east-1

# 2. Enable static website hosting
aws s3 website s3://your-portfolio-name \
  --index-document index.html \
  --error-document index.html

# 3. Upload files
aws s3 sync . s3://your-portfolio-name --exclude "*.md"
```

In the AWS Console, go to **S3 → Permissions → Block public access → Uncheck all → Save**, then attach this bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-portfolio-name/*"
    }
  ]
}
```

Live URL:
```
http://your-portfolio-name.s3-website-us-east-1.amazonaws.com
```

#### Option B — Azure Blob Storage

```bash
# 1. Create storage account
az storage account create \
  --name yourportfolioname \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

# 2. Enable static website hosting
az storage blob service-properties update \
  --account-name yourportfolioname \
  --static-website \
  --index-document index.html \
  --404-document index.html

# 3. Upload to $web container
az storage blob upload-batch \
  --account-name yourportfolioname \
  --source . \
  --destination '$web'

# 4. Get live URL
az storage account show \
  --name yourportfolioname \
  --query "primaryEndpoints.web" \
  --output tsv
```

#### Optional — Add a CDN

**AWS CloudFront:**
1. Go to **CloudFront → Create Distribution**
2. Set origin to your S3 website endpoint
3. Enable **Redirect HTTP to HTTPS**
4. Deploy → get a `*.cloudfront.net` URL

**Azure CDN:**
```bash
az cdn profile create --name myPortfolioCDN \
  --resource-group myResourceGroup --sku Standard_Microsoft

az cdn endpoint create --name myPortfolioEndpoint \
  --profile-name myPortfolioCDN \
  --resource-group myResourceGroup \
  --origin yourportfolioname.z13.web.core.windows.net
```

### Project Structure

```
project-3-global-launch/
├── index.html       # Portfolio page (HTML/CSS/JS — single file)
├── resume.pdf       # Downloadable CV
└── README.md
```

### Cost Estimate

| Service | Free Tier | Beyond Free Tier |
|---|---|---|
| AWS S3 | 5 GB + 20K GET/mo | ~$0.023/GB/mo |
| AWS CloudFront | 1 TB transfer/mo | ~$0.0085/GB |
| Azure Blob Storage | 5 GB + 10K operations | ~$0.018/GB/mo |

> For a personal portfolio with moderate traffic, **monthly cost is effectively $0**.

### Key Concepts

- **Static website hosting** — pre-built HTML/CSS/JS, no server-side rendering
- **Object storage** — files stored as objects in a bucket, served over HTTP
- **Bucket policy** — IAM JSON rule granting public `s3:GetObject` access
- **CDN** — caches the site at global edge locations for low-latency delivery
- **Serverless hosting** — no EC2/VM provisioned; storage handles all HTTP traffic

---

<br/>

## 🔁 Projects Comparison

| | Project 2 — Server Commander | Project 3 — Global Launch |
|---|---|---|
| **Infrastructure** | EC2 Virtual Machine | S3 / Blob Storage (no server) |
| **OS access** | Full (Ubuntu root) | None |
| **Scaling** | Manual (instance resize) | Automatic (infinite) |
| **Cost model** | Per-hour instance billing | Per-request/GB storage billing |
| **Use case** | Dynamic apps, custom software | Static sites, portfolios, SPAs |
| **Setup complexity** | High (SSH, firewall, Nginx) | Low (upload + policy) |
| **Security surface** | Large (open ports, SSH keys) | Minimal (bucket policy only) |

---

## 👤 Author

**Badar Ali**  
Software Engineering Student — FAST NUCES Karachi  
Full-Stack & Backend Developer  
Cloud Computing Internship · DecodeLabs

---

## 📄 License

MIT — free to use, modify, and deploy.

<br/>

## 🖥️ Project 2 — The Server Commander

> **Scenario:** A startup needs a dedicated server environment with full OS control to install custom software and security patches.

### Overview

Acting as a SysAdmin, this project covers provisioning a cloud-based virtual server from scratch — launching an EC2 instance, securing it via SSH, installing a web server, and deploying a custom webpage. No managed services, no GUIs — just the terminal.

### Tech Stack

| Layer | Technology |
|---|---|
| Cloud Provider | AWS EC2 |
| OS | Ubuntu 22.04 LTS |
| Web Server | Nginx 1.24 |
| Access | SSH (key-pair authentication) |
| Security | AWS Security Groups + UFW Firewall |

### Step-by-Step Setup

#### Step 1 — Launch EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**
2. Configure:
   - **Name:** `decodelabs-server`
   - **AMI:** Ubuntu Server 22.04 LTS (Free Tier eligible)
   - **Instance Type:** `t2.micro`
   - **Key Pair:** Create new → download `.pem` file
   - **Security Group:** Allow **SSH (22)**, **HTTP (80)**, **HTTPS (443)**
3. Click **Launch Instance**

#### Step 2 — Connect via SSH

```bash
# Fix permissions on your key file (required on Linux/Mac)
chmod 400 your-key.pem

# Connect to your instance
ssh -i "your-key.pem" ubuntu@<YOUR_EC2_PUBLIC_IP>
```

#### Step 3 — Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

#### Step 4 — Install Nginx

```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

Expected output:
```
● nginx.service - A high performance web server and a reverse proxy server
   Active: active (running) since ...
```

#### Step 5 — Configure Firewall (UFW)

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

#### Step 6 — Deploy the Custom Webpage

```bash
cd /var/www/html
sudo rm index.nginx-debian.html
sudo nano index.html
# Paste index.html contents, then save (Ctrl+O → Enter → Ctrl+X)

sudo chown -R www-data:www-data /var/www/html
sudo systemctl reload nginx
```

#### Step 7 — Verify

Navigate to `http://<YOUR_EC2_PUBLIC_IP>` — the **Welcome to DecodeLabs** page should be live. ✅

### Project Structure

```
project-2-server-commander/
├── index.html       # Custom "Welcome to DecodeLabs" webpage
└── README.md
```

### Security Notes

- SSH restricted to key-pair auth only (password login disabled)
- AWS Security Groups as the first firewall layer
- UFW handles OS-level traffic filtering
- Only ports 22, 80, and 443 exposed to the internet

### Key Concepts

- **Cloud provisioning** — launching and configuring a VM on AWS
- **SSH security** — key-pair auth and secure remote access
- **Linux CLI** — package management, file permissions, service control
- **Web server config** — installing, enabling, and verifying Nginx
- **Layered firewall** — Security Groups + UFW working together

---

<br/>
