# Static Website Hosting with S3, CloudFront & Route 53

A secure, fast static website hosted on AWS. Content is stored in a private **S3** bucket and delivered globally through a **CloudFront** CDN, with **HTTPS** from **ACM**, a custom domain in **Route 53**, and access controlled through **IAM** and **bucket policies**.

---

## Architecture

```
User ──► Route 53 (custom domain)
              │
              ▼
        CloudFront (CDN, HTTPS via ACM certificate)
              │   Origin Access Control (OAC)
              ▼
        S3 bucket (private, static website files)

        CloudWatch ──► monitors CloudFront / S3 metrics
```

> Add your architecture diagram here: `![Architecture](images/architecture.png)`

---

## AWS Services Used

| Service | Purpose |
|---|---|
| **Amazon S3** | Stores the static website files (HTML, CSS, JS, images) |
| **Amazon CloudFront** | CDN for fast, global content delivery and HTTPS |
| **Amazon Route 53** | DNS and custom domain mapping |
| **AWS Certificate Manager (ACM)** | SSL/TLS certificate for HTTPS |
| **AWS IAM** | Controls who can access and manage AWS resources |
| **S3 Bucket Policy** | Restricts bucket access to CloudFront only |
| **Amazon CloudWatch** | Monitoring of metrics and logs |

---

## Key Features

- **Secure:** the S3 bucket is private; only CloudFront can read from it.
- **HTTPS enabled:** SSL/TLS certificate issued and managed by ACM.
- **Fast:** content is cached at CloudFront edge locations worldwide.
- **Custom domain:** DNS records managed in Route 53.
- **Least-privilege access:** IAM and bucket policies limit permissions.
- **Monitored:** CloudWatch metrics for visibility into traffic and errors.

---

## Setup Steps

### 1. Create the S3 bucket
1. Open the S3 console and create a bucket (for example, `your-domain-name-site`).
2. Keep **Block all public access** turned **on**.
3. Upload your website files (`index.html`, `error.html`, assets).

### 2. Request an SSL certificate (ACM)
1. Open ACM in the **us-east-1 (N. Virginia)** region. CloudFront requires certificates from this region.
2. Request a public certificate for your domain (and `www` if needed).
3. Choose **DNS validation** and add the CNAME record in Route 53.
4. Wait until the status shows **Issued**.

### 3. Create the CloudFront distribution
1. Set the S3 bucket as the **origin**.
2. Use **Origin Access Control (OAC)** so only CloudFront can access the bucket.
3. Set **Viewer protocol policy** to *Redirect HTTP to HTTPS*.
4. Add your domain as an **Alternate domain name (CNAME)** and attach the ACM certificate.
5. Set the **Default root object** to `index.html`.

### 4. Apply the S3 bucket policy
Copy the policy that CloudFront generates for OAC and paste it into the bucket's **Permissions → Bucket policy**. It should look like this (replace the placeholders):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::YOUR-ACCOUNT-ID:distribution/YOUR-DISTRIBUTION-ID"
        }
      }
    }
  ]
}
```

### 5. Configure Route 53
1. Open your hosted zone.
2. Create an **A record (Alias)** pointing your domain to the CloudFront distribution.
3. Repeat for `www` if needed.

### 6. Test
Open `https://your-domain.com` and confirm the site loads over HTTPS.

---

## Security Considerations

- S3 bucket is **not public**; access is only through CloudFront (OAC).
- HTTP requests are **redirected to HTTPS**.
- IAM users and roles follow the **principle of least privilege**.
- MFA is recommended for the AWS account root and admin users.

---

## Monitoring

Amazon CloudWatch is used to track CloudFront and S3 metrics such as requests, error rates, and data transfer.

---

## Project Structure

```
├── index.html
├── error.html
├── css/
├── js/
├── images/
└── README.md
```

---

## What I Learned

- Hosting and securing static content with S3 and CloudFront
- Using Origin Access Control instead of making a bucket public
- Issuing and validating SSL certificates with ACM
- Managing DNS records in Route 53
- Writing bucket policies and applying least-privilege access

---

## Future Improvements

- Automate the deployment with **CloudFormation** or **Terraform**
- Add a CI/CD pipeline to upload changes to S3 and invalidate the CloudFront cache
- Add AWS WAF for extra protection

---

## Author

**Divya Jadhav**
AWS Cloud Engineer (Fresher)
[LinkedIn](https://www.linkedin.com/in/divya-jadhav-4555752a6/) | [GitHub](https://github.com/divyajadhav111-del) | divyajadhav1011@gmail.com
