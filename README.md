# AWS S3 CORS Demonstration Guide

Step-by-step practical guide to demonstrate AWS S3 Cross-Origin Resource Sharing (CORS) using two S3 buckets with Static Web Hosting enabled.

[![Deploy to Zerops](http://localhost:3000/badge)](http://localhost:3000/deploy?repo=Amitabh-DevOps%2Faws-s3-cors)


---

## 📁 Repository Files

| File | Upload To |
| :--- | :--- |
| `index.html` | Bucket 1 (Origin Bucket) |
| `styles.css` | Bucket 1 (Origin Bucket) |
| `coffee.jpg` | Bucket 1 (Origin Bucket) |
| `extra-page.html` | Bucket 2 (Resource Bucket) |
| `extra-styles.css` | Bucket 2 (Resource Bucket) |
| `cors-policy.json` | Apply to Bucket 2 CORS settings |

---

## Step 1: Set up Bucket 1 (Origin Bucket)

1. Create Bucket 1 (e.g. `s3-cross-origin-bucket-981526069311-us-east-1-an`).
2. Enable **Static Website Hosting** (Index document: `index.html`).
3. Add public read **Bucket Policy**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::s3-cross-origin-bucket-981526069311-us-east-1-an/*"
       }
     ]
   }
   ```
4. Upload `index.html`, `styles.css`, and `coffee.jpg` to Bucket 1.

---

## Step 2: Set up Bucket 2 (Resource Bucket)

1. Create Bucket 2 (e.g. `s3-cross-origin-other-bucket-981526069311-eu-west-1-an`).
2. Enable **Static Website Hosting** (Index document: `extra-page.html`).
3. Add public read **Bucket Policy**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::s3-cross-origin-other-bucket-981526069311-eu-west-1-an/*"
       }
     ]
   }
   ```
4. Upload `extra-page.html` and `extra-styles.css` to Bucket 2.

---

## Step 3: Reproduce the CORS Error (Without CORS) 🔴

1. Open Bucket 1 Static Website URL in your browser:  
   `http://s3-cross-origin-bucket-981526069311-us-east-1-an.s3-website-us-east-1.amazonaws.com`
2. Open **Browser DevTools (F12) → Console tab**.
3. Observe the CORS error:
   ```text
   Access to fetch at 'http://s3-cross-origin-other-bucket-981526069311-eu-west-1-an.s3-website-eu-west-1.amazonaws.com/extra-page.html' 
   from origin 'http://s3-cross-origin-bucket-981526069311-us-east-1-an.s3-website-us-east-1.amazonaws.com' 
   has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
   ```
4. The `<div id="tofetch"></div>` remains empty on the page.

---

## Step 4: Enable CORS on Bucket 2 (The Fix) 🟢

1. Go to AWS S3 Console → Select **Bucket 2**.
2. Click **Permissions** tab → scroll to **Cross-origin resource sharing (CORS)** → click **Edit**.
3. Paste the contents of `cors-policy.json`:
   ```json
   [
       {
           "AllowedHeaders": ["Authorization"],
           "AllowedMethods": ["GET"],
           "AllowedOrigins": [
               "http://s3-cross-origin-bucket-981526069311-us-east-1-an.s3-website-us-east-1.amazonaws.com"
           ],
           "ExposeHeaders": [],
           "MaxAgeSeconds": 3000
       }
   ]
   ```
4. Click **Save changes**.
5. Refresh the Bucket 1 webpage.
6. **Result:** 🟢 The content of `extra-page.html` is fetched and rendered inside the page!
