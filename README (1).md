
# Sample Cross-Account Deployment (AWS)

This repository demonstrates a simple and secure **cross-AWS-account deployment** using **AWS CodePipeline** and **CodeBuild**.

The pipeline runs in one AWS account (CI / Management account) and deploys artifacts to another AWS account (Target account) using **IAM role assumption (STS)**.

⚠️ **This repository is for learning and testing purposes only.**

---

## Architecture Overview

```
GitHub Repository
       |
CodePipeline (CI Account – ACG Sandbox)
       |
CodeBuild
       |
STS AssumeRole
       |
S3 Bucket (Target Account – Personal AWS)
```

---

## What This Project Does

- Uses **CodePipeline** to trigger on GitHub commits
- Uses **CodeBuild** to assume a role in another AWS account
- Deploys a static HTML file (`index.html`) to an **S3 bucket** in a different AWS account
- Demonstrates real cross-account access **without sharing credentials**

---

## Repository Structure

```
.
├── index.html        # Sample static website file
├── buildspec.yml     # CodeBuild instructions
└── README.md         # Documentation
```

---

## Prerequisites

1. Two AWS accounts:
   - **Account A (CI Account)** → A Cloud Guru Sandbox
   - **Account B (Target Account)** → Personal AWS account
2. GitHub repository connected to CodePipeline
3. AWS CLI access (for validation/testing)

---

## Step-by-Step Setup

### STEP 1: Create S3 Bucket in Target Account (Account B)

1. Login to Personal AWS Account
2. Go to **S3 → Create bucket**
3. Bucket name example:  
   ```
   cross-account-deploy-demo-bucket
   ```
4. Leave defaults and create the bucket  

> This bucket will receive `index.html` from the pipeline.

---

### STEP 2: Create Cross-Account IAM Role (Account B)

This role allows the CI account to deploy into the target account.

#### 2.1 Create Role

- Go to **IAM → Roles → Create role**
- Trusted entity: **AWS Account**
- Account ID: CI (Sandbox) Account ID

#### 2.2 Trust Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<CI_ACCOUNT_ID>:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### 2.3 Permissions Policy (S3 Only – Least Privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::cross-account-deploy-demo-bucket",
        "arn:aws:s3:::cross-account-deploy-demo-bucket/*"
      ]
    }
  ]
}
```

- **Role name:** `CrossAccountS3DeployRole`  
- Copy the **Role ARN** (used later)

---

### STEP 3: Create CodeBuild Service Role (Account A – CI)

1. Login to ACG Sandbox AWS account
2. Go to **IAM → Roles → Create role**
3. Trusted entity: **AWS Service → CodeBuild**
4. Permissions Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<TARGET_ACCOUNT_ID>:role/CrossAccountS3DeployRole"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:*",
        "s3:GetObject"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### STEP 4: Create CodeBuild Project (Account A)

1. Go to **CodeBuild → Create project**
2. Source: **GitHub** (same repo)
3. Environment:
   - Managed image
   - Amazon Linux 2
4. Service role: **Select the role created above**

#### `buildspec.yml`

```yaml
version: 0.2

phases:
  build:
    commands:
      - echo "Assuming cross-account role"
      - aws sts assume-role           --role-arn arn:aws:iam::<TARGET_ACCOUNT_ID>:role/CrossAccountS3DeployRole           --role-session-name simple-cross-account > creds.json

      - export AWS_ACCESS_KEY_ID=$(jq -r '.Credentials.AccessKeyId' creds.json)
      - export AWS_SECRET_ACCESS_KEY=$(jq -r '.Credentials.SecretAccessKey' creds.json)
      - export AWS_SESSION_TOKEN=$(jq -r '.Credentials.SessionToken' creds.json)

      - echo "Uploading index.html to target account S3"
      - aws s3 cp index.html s3://cross-account-deploy-demo-bucket/index.html
```

---

### STEP 5: Create CodePipeline (Account A)

1. Go to **CodePipeline → Create pipeline**
2. Pipeline role: **default or custom**
3. Artifact store: **S3 (same account)**
4. Stages:
   - **Source:** GitHub
   - **Build:** CodeBuild project created above
   - **No deploy stage needed** (handled by CodeBuild)

---

### STEP 6: Test the Deployment

1. Modify `index.html`
2. Commit and push to GitHub
3. Pipeline triggers automatically
4. Verify file upload in Target Account S3 bucket
5. Open object URL to view the webpage

---

## Security Notes

- ✅ No access keys are shared
- ✅ Uses temporary credentials via STS
- ✅ Follows least privilege principle
- ✅ Safe for sandbox and practice environments

---

## Common Errors & Fixes

| Issue                         | Fix                                  |
|--------------------------------|-------------------------------------|
| AccessDenied on AssumeRole      | Check trust policy account ID       |
| S3 upload denied                | Check bucket ARN in policy          |
| Pipeline fails                  | Check CodeBuild logs                |

---

## Interview Explanation (Short)

> “I configured CodePipeline in one AWS account and deployed a static site to another AWS account using STS AssumeRole. This ensured secure cross-account access without sharing credentials.”

---

## Disclaimer

This project is for **learning and demonstration purposes only**.  
Do not use these configurations directly in production without a security review.
