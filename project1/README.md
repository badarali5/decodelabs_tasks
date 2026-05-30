# 🌍 The Global Launch — Static Portfolio on Cloud Storage

> A serverless personal portfolio hosted on AWS S3 / Azure Blob Storage.  
> No backend. No provisioning. Instant global access.

---

## 📌 Project Overview

This project demonstrates how to host a **static portfolio website** using cloud object storage — without provisioning a single server. The site loads instantly worldwide, costs near zero, and scales infinitely.

**Tools used:** AWS S3 · Azure Blob Storage · AWS CloudFront (optional CDN)

---

## 📁 Project Structure

```
portfolio/
├── index.html        # Main portfolio page (HTML/CSS/JS — single file)
├── resume.pdf        # Downloadable CV
└── README.md         # This file
```

---

## 🚀 Deployment Guide

### Option A — AWS S3

#### 1. Create an S3 Bucket
```bash
aws s3 mb s3://your-portfolio-name --region us-east-1
```
> Bucket name must be globally unique.

#### 2. Disable Public Access Block
In the AWS Console:  
**S3 → Your Bucket → Permissions → Block public access → Edit → Uncheck all → Save**

#### 3. Enable Static Website Hosting
```bash
aws s3 website s3://your-portfolio-name \
  --index-document index.html \
  --error-document index.html
```

#### 4. Attach a Public Read Policy
Go to **Permissions → Bucket Policy** and paste:
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

#### 5. Upload Files
```bash
aws s3 sync . s3://your-portfolio-name --exclude "*.md"
```

#### 6. Access Your Site
```
http://your-portfolio-name.s3-website-us-east-1.amazonaws.com
```

---

### Option B — Azure Blob Storage

#### 1. Create a Storage Account
```bash
az storage account create \
  --name yourportfolioname \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2
```

#### 2. Enable Static Website Hosting
```bash
az storage blob service-properties update \
  --account-name yourportfolioname \
  --static-website \
  --index-document index.html \
  --404-document index.html
```

#### 3. Upload Files to `$web` Container
```bash
az storage blob upload-batch \
  --account-name yourportfolioname \
  --source . \
  --destination '$web'
```

#### 4. Get Your Live URL
```bash
az storage account show \
  --name yourportfolioname \
  --query "primaryEndpoints.web" \
  --output tsv
```
Output example:
```
https://yourportfolioname.z13.web.core.windows.net/
```

---

## ⚡ Optional — Add a CDN (Recommended)

Wrap your bucket with a CDN to get HTTPS + edge caching globally.

**AWS CloudFront:**
1. Go to **CloudFront → Create Distribution**
2. Set origin to your S3 website endpoint
3. Enable **Redirect HTTP to HTTPS**
4. Deploy — you get a `*.cloudfront.net` URL

**Azure CDN:**
```bash
az cdn profile create --name myPortfolioCDN \
  --resource-group myResourceGroup --sku Standard_Microsoft

az cdn endpoint create --name myPortfolioEndpoint \
  --profile-name myPortfolioCDN \
  --resource-group myResourceGroup \
  --origin yourportfolioname.z13.web.core.windows.net
```

---

## 💰 Cost Estimate

| Service | Free Tier | Beyond Free Tier |
|---|---|---|
| AWS S3 | 5 GB storage + 20K GET/mo | ~$0.023/GB/mo |
| AWS CloudFront | 1 TB transfer/mo | ~$0.0085/GB |
| Azure Blob Storage | 5 GB + 10K operations | ~$0.018/GB/mo |

> For a personal portfolio with moderate traffic, **monthly cost is effectively $0**.

---

## 🧠 Key Concepts

| Concept | Explanation |
|---|---|
| **Static Website** | Pre-built HTML/CSS/JS — no server-side rendering needed |
| **Object Storage** | Files stored as objects in a bucket, served over HTTP |
| **Bucket Policy** | IAM JSON rule granting public `s3:GetObject` access |
| **CDN** | Content Delivery Network — caches site at global edge locations |
| **Serverless Hosting** | No EC2/VM provisioned; storage service handles all HTTP traffic |

---

## ✏️ Customization

Before deploying, update these fields in `index.html`:

- `badar@example.com` → your real email  
- `github.com/badar` → your GitHub profile URL  
- `linkedin.com/in/badar` → your LinkedIn URL  
- Replace `resume.pdf` with your actual resume file  

---

## 👤 Author

**Badar Abbas**  
Software Engineering Student — FAST NUCES Karachi  
Full-Stack & Backend Developer  

---

## 📄 License

MIT — free to use, modify, and deploy.
