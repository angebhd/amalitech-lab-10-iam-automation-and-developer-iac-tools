# IAM Testing – IAM Permissions via AWS CLI

## Lab 2 – Manual IAM Management via AWS CLI

### Overview

Create and test IAM permissions manually using AWS CloudShell and a locally configured AWS CLI profile. The lab demonstrates permission boundaries by verifying access and intentional denial across S3 and EC2.

### Prerequisites

- AWS account with sufficient privileges to manage IAM
- AWS CLI installed on your local machine
- AWS CloudShell access (available in the AWS Console top-right toolbar)

---

### Task 1 (50%) – Create IAM User and Attach S3 Permissions

**Step 1: Create the IAM user (CloudShell)**

```bash
aws iam create-user --user-name lab-user
```

**Step 2: Create access keys**

```bash
aws iam create-access-key --user-name lab-user
```

Save the `AccessKeyId` and `SecretAccessKey` — you will need them in Task 2.

**Step 3: Attach the S3 managed policy**

```bash
aws iam attach-user-policy \
  --user-name lab-user \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

---

### Task 2 (25%) – Configure and Test AWS CLI Locally

**Step 1: Configure a named profile**

```bash
aws configure --profile lab-user
```

Enter your Access Key, Secret Key, region (e.g. `us-east-1`), and output format (`json`).

**Step 2: Verify the configured identity**

```bash
aws sts get-caller-identity --profile lab-user
```

**Step 3: List existing S3 buckets**

```bash
aws s3 ls --profile lab-user
```

**Step 4: Create two S3 buckets**

> Bucket names must be globally unique.

```bash
aws s3 mb s3://<your-bucket-name-1> --region us-east-1 --profile lab-user
aws s3 mb s3://<your-bucket-name-2> --region us-east-1 --profile lab-user
```

**Step 5: Confirm EC2 access is denied**

```bash
aws ec2 describe-instances --profile lab-user
```

Expected result: `AccessDenied` error — the user has no EC2 permissions yet.

---

### Task 3 (25%) – Grant and Verify EC2 Read Access

**Step 1: Attach the EC2 read-only policy (CloudShell)**

```bash
aws iam attach-user-policy \
  --user-name lab-user \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
```

**Step 2: Re-run the EC2 command locally**

```bash
aws ec2 describe-instances --profile lab-user
```

Expected result: success — instances listed without error.

---

### Screenshot Checklist

| File | Content |
|---|---|
| `Full_Name_Image_1` | IAM user creation output |
| `Full_Name_Image_2` | Access keys created + S3 policy attached |
| `Full_Name_Image_3` | AWS CLI profile configuration |
| `Full_Name_Image_4` | `sts get-caller-identity` output |
| `Full_Name_Image_5` | S3 bucket listing |
| `Full_Name_Image_6` | Two S3 buckets created |
| `Full_Name_Image_7` | EC2 `AccessDenied` error |
| `Full_Name_Image_8` | EC2 access successful after policy attachment |

> Always capture both the command and its output in each screenshot.

---

## Cleanup

AWS will not allow deleting a user until all attached resources are removed. Run the following in CloudShell:

```bash
# Remove access key
aws iam delete-access-key \
  --user-name lab-user \
  --access-key-id $(aws iam list-access-keys --user-name lab-user \
    --query 'AccessKeyMetadata[0].AccessKeyId' --output text)

# Detach policies
aws iam detach-user-policy \
  --user-name lab-user \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

aws iam detach-user-policy \
  --user-name lab-user \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

# Delete login profile if created
aws iam delete-login-profile --user-name lab-user

# Delete the user
aws iam delete-user --user-name lab-user
```

After deletion, remove the local credentials profile:

```bash
nano ~/.aws/credentials  # Delete the [lab-user] section
```

> **Never commit AWS credentials to this repository.**
