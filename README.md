# AWS VPC Flow Logs to S3

This project sets up **VPC Flow Logs** for a specified **AWS VPC** and delivers the logs to an **Amazon S3** bucket using an **IAM Role**. It helps monitor network traffic for security, auditing, and performance analysis.

## 📁 Repository

GitHub: [https://github.com/kunal7703/AWS-Logs-Flow-To-S3](https://github.com/kunal7703/AWS-Logs-Flow-To-S3)

---

## 📌 Project Objective

To configure VPC Flow Logs for a selected VPC, store logs securely in an S3 bucket using a role-based access mechanism, and ensure logs can be used for audit and analysis.

---

## 📋 Project Steps

### 1️⃣ Create/Identify a VPC
- Use an existing VPC or create a new one via the AWS Console or CLI.

### 2️⃣ Create an S3 Bucket
- Bucket name: `vpc-flow-logs-bucket` (or your preferred name)
- Enable default encryption and versioning (optional but recommended)

### 3️⃣ Apply S3 Bucket Policy

File: `s3-bucket-policy.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VPCFlowLogsDelivery",
      "Effect": "Allow",
      "Principal": {
        "Service": "vpc-flow-logs.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::vpc-flow-logs-bucket/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceAccount": "<account-id>"
        }
      }
    }
  ]
}
Replace <account-id> with your AWS Account ID.

4️⃣ Create IAM Role for VPC Flow Logs
a. Trust Relationship
File: trust-policy.json

json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vpc-flow-logs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
b. Permissions Policy
File: iam-role-policy.json

json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::vpc-flow-logs-bucket/*"
    }
  ]
}
5️⃣ Create the Role in AWS
bash
Copy
Edit
aws iam create-role --role-name VPCFlowLogsRole \
  --assume-role-policy-document file://trust-policy.json

aws iam put-role-policy --role-name VPCFlowLogsRole \
  --policy-name VPCFlowLogsS3Policy \
  --policy-document file://iam-role-policy.json
6️⃣ Enable VPC Flow Logs
Go to the VPC console > Flow Logs tab.

Click Create Flow Log.

Choose:

Resource Type: VPC

Traffic Type: ALL (for full visibility)

Destination: Send to an S3 bucket

Destination bucket ARN: arn:aws:s3:::vpc-flow-logs-bucket

IAM Role: VPCFlowLogsRole

✅ Verification Steps
Launch an EC2 instance inside the VPC and generate some network traffic (e.g., ping, curl).

Check your S3 bucket path:

php-template
Copy
Edit
s3://vpc-flow-logs-bucket/<account-id>/<region>/<vpc-id>/
Open a log file. Sample log line:

Copy
Edit
eni-abc123456 10.0.1.12 10.0.1.45 443 34567 6 10 840 1625581001 1625581061 ACCEPT OK
Log Fields Explained:
eni-abc123456: Network Interface ID

10.0.1.12: Source IP

10.0.1.45: Destination IP

443: Destination port (HTTPS)

6: Protocol (TCP)

ACCEPT: Flow result

OK: Delivery status

📂 Folder Structure
pgsql
Copy
Edit
AWS-Logs-Flow-To-S3/
├── iam-role-policy.json
├── trust-policy.json
├── s3-bucket-policy.json
├── screenshots/
│   ├── flow-log-setup.png
│   ├── s3-bucket-logs.png
│   └── sample-log.png
└── README.md
🔐 Why Use IAM Role?
Limits access to only log delivery service

No credentials involved (uses service-to-service trust)

Logs are securely written to S3

📊 Traffic Type Chosen: ALL
This includes both accepted and rejected traffic and gives the most complete visibility for auditing, troubleshooting, and compliance needs.

🙌 Conclusion
With this setup, VPC Flow Logs are securely and automatically stored in S3, making it easy to analyze traffic, investigate anomalies, and meet compliance requirements.
