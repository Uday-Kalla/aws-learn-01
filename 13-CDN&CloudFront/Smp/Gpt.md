> Here’s a **step-by-step guide to configure a sample static website using AWS CloudFront** (ideal for hosting things like HTML/CSS/JS sites, or static frontend apps like React):

## 🧭 Overview:

You will:

1. Upload website files to an S3 bucket.
2. Make the S3 bucket public (or private + signed URL if needed).
3. Create a CloudFront distribution pointing to the S3 bucket.
4. Test your website via the CloudFront URL.

---

## ✅ Prerequisites

* AWS account
* Website files (e.g., `index.html`, `style.css`, etc.)
* AWS CLI installed (optional but helpful)

---

## 🚀 Step-by-Step Setup

### **Step 1: Create an S3 Bucket**

1. Go to **S3 Console** → Click **“Create bucket”**
2. Give it a unique name like `my-sample-static-site`
3. **Uncheck** “Block all public access” (if public site)
4. Enable versioning (optional)
5. Create the bucket

---

### **Step 2: Upload Website Files**

1. Click the bucket name → “Upload”
2. Add your files (`index.html`, etc.)
3. Click “Upload”

---

### **Step 3: Enable Static Website Hosting**

1. Go to **“Properties”** tab of your bucket
2. Scroll down to **“Static website hosting”**
3. Click **“Edit”** → Enable it
4. Set:

   * **Index document**: `index.html`
   * (Optional) **Error document**: `error.html`
5. Save changes

---

### **Step 4: Make Files Public (if required)**

If public access is needed:

1. Go to “Permissions” → **Bucket policy**
2. Add this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadForStaticWebsite",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-sample-static-site/*"
    }
  ]
}
```

> 🔁 Replace `my-sample-static-site` with your bucket name.

---

### **Step 5: Create CloudFront Distribution**

1. Go to **CloudFront Console** → Click **“Create Distribution”**
2. Under **Origin domain**, select your S3 bucket (or paste website endpoint)
3. **Origin access**:

   * If your bucket is private → choose **Origin access control (OAC)** and configure it
   * If your bucket is public → you can skip OAC
4. Under **Default cache behavior settings**:

   * Viewer Protocol Policy: **Redirect HTTP to HTTPS**
   * Allowed HTTP methods: **GET, HEAD**
5. Under **Default root object**, enter `index.html`
6. Click **Create distribution**

---

### **Step 6: Wait for Distribution Deployment**

* Takes around **5–15 minutes**
* After it's ready, go to the **CloudFront URL** like:

  ```
  https://d1234abc5678.cloudfront.net/
  ```

---

## 🧪 Step 7: Test the Website

* Open the CloudFront URL in a browser
* You should see your website's homepage

---

## 🔧 Optional (Advanced)

* Use **custom domain** with Route53
* Add **SSL certificate** via AWS Certificate Manager
* Set **cache invalidation** when you update content

---

## 📝 Example Folder Structure

```
project-folder/
├── index.html
├── about.html
└── css/
    └── style.css
```

### **<p align="center"> Terraform Implimentation</p>**
---

> Would you like a **Terraform version** of this setup or integration with **CI/CD like GitHub Actions** too?

---
Great! Here's the **Terraform configuration + GitHub Actions CI/CD** to deploy a static website using **S3 + CloudFront**.

---

## ✅ Structure

```
static-website-project/
├── site/                 # Your website files (HTML, CSS, JS)
│   └── index.html
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── .github/workflows/
│   └── deploy.yml
```

---

## 📁 1. `terraform/main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "static_site" {
  bucket = var.bucket_name
  force_destroy = true
}

resource "aws_s3_bucket_website_configuration" "static_site" {
  bucket = aws_s3_bucket.static_site.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "index.html"
  }
}

resource "aws_s3_bucket_policy" "public_read" {
  bucket = aws_s3_bucket.static_site.id

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Sid       = "PublicReadGetObject",
        Effect    = "Allow",
        Principal = "*",
        Action    = "s3:GetObject",
        Resource  = "${aws_s3_bucket.static_site.arn}/*"
      }
    ]
  })
}

resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.static_site.website_endpoint
    origin_id   = "S3-${aws_s3_bucket.static_site.id}"

    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "http-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }

  enabled             = true
  default_root_object = "index.html"

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.static_site.id}"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "redirect-to-https"
  }

  price_class = "PriceClass_100"

  viewer_certificate {
    cloudfront_default_certificate = true
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
}
```

---

## 📁 2. `terraform/variables.tf`

```hcl
variable "bucket_name" {
  description = "Name of the S3 bucket"
  default     = "my-static-website-example-bucket"
}
```

---

## 📁 3. `terraform/outputs.tf`

```hcl
output "cloudfront_url" {
  value = aws_cloudfront_distribution.cdn.domain_name
}
```

---

## 🔁 4. GitHub Actions CI/CD: `.github/workflows/deploy.yml`

```yaml
name: Deploy to S3 and CloudFront

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Install Terraform
      uses: hashicorp/setup-terraform@v2

    - name: Terraform Init
      working-directory: terraform
      run: terraform init

    - name: Terraform Apply
      working-directory: terraform
      run: terraform apply -auto-approve

    - name: Upload website files to S3
      run: |
        aws s3 sync site/ s3://${{ secrets.S3_BUCKET_NAME }} --delete
```

---

## 🔐 GitHub Secrets Needed

Go to **GitHub repo → Settings → Secrets → Actions** and add:

| Name                    | Value                             |
| ----------------------- | --------------------------------- |
| `AWS_ACCESS_KEY_ID`     | Your AWS access key               |
| `AWS_SECRET_ACCESS_KEY` | Your AWS secret key               |
| `S3_BUCKET_NAME`        | The bucket name (same as in vars) |

---

## ✅ Final Steps

1. Run `terraform init && terraform apply` in `terraform/` once locally
2. Push code to `main` → CI/CD will deploy
3. Access your site via the CloudFront URL from Terraform output

---

Would you like a **PDF guide** of this full setup with visuals too?
