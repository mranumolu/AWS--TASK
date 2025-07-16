# AWS S3 Triggered Lambda Implementation Guide

This guide provides detailed step-by-step instructions for setting up an AWS Lambda function triggered by S3 file uploads. The Lambda runs in a custom VPC with three subnets and prints a random message each time it is triggered.

---

## 1. Create a VPC with 3 Subnets

### 1.1 Open the VPC Dashboard

- Go to [AWS Console](https://console.aws.amazon.com/).
- In the top search bar, type `VPC` and select **VPC**.

### 1.2 Create the VPC

- Click **Create VPC**.
- **Name:** `backend-vpc`
- **IPv4 CIDR block:** `10.0.0.0/16`
- Leave other options as default.
- Click **Create VPC**.

### 1.3 Create Three Subnets

- In the VPC console, click **Subnets**.
- Click **Create subnet**.
- **VPC:** Select `backend-vpc`
- Add the following subnets one at a time:

| Subnet Name | Availability Zone | IPv4 CIDR Block |
|-------------|-------------------|-----------------|
| backend-1a  | us-east-1a        | 10.0.1.0/24     |
| backend-1b  | us-east-1b        | 10.0.2.0/24     |
| backend-1c  | us-east-1c        | 10.0.3.0/24     |

- After all are added, click **Create subnet**.

---

## 2. Create a Lambda Function (Python)

### 2.1 Open Lambda Console

- In AWS Console, search for **Lambda**, and select it.

### 2.2 Create the Lambda Function

- Click **Create function**.
- **Author from scratch**:
  - **Function name:** `S3UploadTriggerLambda`
  - **Runtime:** Python 3.12 (or latest)
  - **Execution role:** Create a new role with basic Lambda permissions (default)
- Click **Create function**.

### 2.3 Attach Lambda to the VPC

- In the function's page, go to **Configuration** > **VPC**.
- Click **Edit**.
- **VPC:** Select `backend-vpc`
- **Subnets:** Select `backend-1b` and `backend-1c` only.
- **Security group:** Default is fine.
- Click **Save**.

### 2.4 Add Python Code

- In the **Code** tab, paste:

    ```python
    import random

    def lambda_handler(event, context):
        messages = [
            "Hello from Lambda!",
            "Random number: " + str(random.randint(1, 100)),
            "S3 triggered this Lambda.",
            "All systems go!",
        ]
        print(random.choice(messages))
        return {"statusCode": 200}
    ```

- Click **Deploy**.

---

## 3. Create an S3 Bucket

### 3.1 Open S3 Console

- Search for and select **S3** in AWS Console.

### 3.2 Create the Bucket

- Click **Create bucket**.
- **Bucket name:** Must be globally unique (e.g., `my-s3-lambda-trigger-<random>`)
- **Region:** Match your Lambda/VPC region.
- Leave other settings as default.
- Click **Create bucket**.

---

## 4. Configure S3 Event Trigger for Lambda

### 4.1 Add S3 Trigger

- Go back to your Lambda function.
- Click **Configuration** > **Triggers**.
- Click **Add trigger**.
- **Select source:** S3
- **Bucket:** Select your bucket
- **Event type:** `All object create events`
- Leave Prefix/Suffix blank.
- **Enable trigger:** Checked.
- Click **Add**.

> **Note:** AWS automatically updates permissions to let S3 invoke this Lambda function.

---

## 5. (Optional) Add S3 Permissions to Lambda Role

If your Lambda needs to **read/write S3 objects**, add S3 permissions:

- Go to **IAM > Roles**.
- Find your Lambda's role (e.g., `lambda-role-S3UploadTriggerLambda-...`)
- Click the role.
- Click **Add permissions** > **Attach policies**.
- Add `AmazonS3ReadOnlyAccess` (for read) or custom as needed.

---

## 6. Test the Integration

### 6.1 Upload a File

- Go to your S3 bucket.
- Click **Upload**, add a file, and click **Upload**.

### 6.2 Check Lambda Output

- Go to **CloudWatch** > **Log groups** > `/aws/lambda/S3UploadTriggerLambda`
- Check the most recent log stream for the random message.

---

## 7. Cleanup (Recommended)

- **Delete the S3 bucket** (delete all files first).
- **Delete the Lambda function**.
- **Delete the VPC and subnets** (if not needed).

---

## Notes for Beginners

- All steps are via AWS Console (web interface).
- Bucket names must be unique globally.
- Permissions for S3 to trigger Lambda are added automatically when you set the trigger.
- Only add S3 IAM permissions to Lambda role if you need to access bucket contents in your code.

---

## Summary Checklist

- [x] VPC: `backend-vpc` with subnets: `backend-1a`, `backend-1b`, `backend-1c`
- [x] Lambda: Python function, attached to `backend-1b` and `backend-1c`
- [x] S3 bucket created
- [x] S3 trigger for Lambda set up
- [x] Test file upload triggers Lambda and logs output

---
