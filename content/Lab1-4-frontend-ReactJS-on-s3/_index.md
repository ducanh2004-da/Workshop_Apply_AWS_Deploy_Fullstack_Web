---

title: "Lab 1 Section IV: ReactJS Frontend on S3"
weight: 4
chapter: false
--------------

# Hosting Your ReactJS Frontend on S3

#### Why It Matters

Delivering your React application as a static website on S3 combines blazing-fast performance with near-zero operational overhead. In this section, you will transform your local React build into a fully hosted, globally available site. You’ll configure an S3 bucket for static hosting, apply fine-grained access controls, and deploy your production-ready artifacts—turning complex CI/CD pipelines into a few straightforward steps.

---

## 1. Create Your S3 Bucket

1. Navigate to **S3 → Buckets → Create bucket**.
2. Configure:

   * **Bucket name**: `webreact2` (must be globally unique).
   * **Region**: choose closest to your users.
   * **Block Public Access**: Uncheck **Block all public access**. Confirm you understand that this bucket will serve public content.
3. (Optional) Add tags such as `Project: Lab-AI` or `Environment: Production` for easy resource tracking.

![Host Fullstack Web A.I](../images/4/4-1.png?featherlight=false&width=90pc)

## 2. Build Your React App Locally

In your project ReactJS directory on localhost, open terminal and run:

```bash
npm run build
```

This command outputs optimized static assets in the `build/` (or `dist/`) folder in project folder, ready for deployment.

## 3. Enable Static Website Hosting

1. In the **S3** console, select your bucket and go to **Properties**.
2. Scroll to **Static website hosting**(last section) and click **Edit**.
3. Choose **Enable** → set **Index document** to `index.html` → click **Save changes**.
4. Note the **Bucket website endpoint** URL—this is the public URL for your React app.

![Host Fullstack Web A.I](../images/4/4-2.png?featherlight=false&width=90pc)

## 4. Configure a Public-Read Bucket Policy

To allow anyone to fetch your static files, apply a bucket policy:

1. Go to **Permissions tab →  edit bucket policy → policy generator**.
2. Type:
- type : S3 bucket policy
- Principal: *
- Actions: GetObject
- ARN: <Bucket ARN> 
=> add statement => generate policy to generate and get JSON
3. Paste the JSON, replacing `webreact2` with your bucket name:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::webreact2/*"
    }
  ]
}
```

3. Click **Save**.

![Host Fullstack Web A.I](../images/4/4-3.png?featherlight=false&width=90pc)

![Host Fullstack Web A.I](../images/4/4-4.png?featherlight=false&width=90pc)

## 5. Upload Your Build Artifacts

1. In the S3 console, open your bucket → **Objects → Upload**.
2. Click **Add files** and **Add folder**, then select all contents of your `build/` directory.
3. Click **Upload**—your React site is now live at the bucket website endpoint.

![Host Fullstack Web A.I](../images/4/4-5.png?featherlight=false&width=90pc)

## 6. Result:

- We switch Properties tab then scroll down and copy the url to run web on browser:

![Host Fullstack Web A.I](../images/4/4-6.png?featherlight=false&width=90pc)

- the data from backend(aws) fetch to frontend(aws) well

![Host Fullstack Web A.I](../images/4/4-7.png?featherlight=false&width=90pc)

## 6. (Optional) Accelerate Delivery with CloudFront

For global caching and HTTPS support, consider fronting your S3 bucket with CloudFront:

1. Create a new CloudFront distribution: Origin = your S3 bucket’s static website endpoint.
2. Configure default cache behavior, SSL certificate (e.g., AWS Certificate Manager), and custom domain if desired.
3. Deploy, then use the CloudFront domain name for fastest global access.

> **Pro Tip:** Automate future deployments by integrating `aws s3 sync build/ s3://webreact2/ --delete` into your CI pipeline. This keeps your bucket in sync with your latest builds without manual uploads.
