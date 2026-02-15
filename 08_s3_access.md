# Access S3 from a VPC

In this phase, I extended my VPC architecture to interact with Amazon S3 using the AWS CLI from within an EC2 instance.

This demonstrates how compute resources inside a VPC can securely access AWS services outside the VPC boundary.
<img width="684" height="394" alt="Screenshot 2026-02-15 at 4 02 30 PM" src="https://github.com/user-attachments/assets/cd520fa0-90a2-48fa-a652-528f2a832a0c" />

---

## Architecture Setup

I created:

- 1 VPC (10.0.0.0/16)
- 1 Public Subnet
- 1 Internet Gateway
- 1 Route Table
- 1 Security Group (SSH only)
- 1 EC2 Instance (Amazon Linux 2023, t2.micro, public IP enabled)

---

## Connecting to EC2

I used **EC2 Instance Connect** to access the instance via SSH directly from the AWS Console.

Once connected, I attempted to run:

```bash
aws s3 ls
```

This initially failed because the EC2 instance had no AWS credentials configured.

---

## Configuring AWS CLI Credentials

To allow the EC2 instance to interact with S3, I:

1. Created an IAM Access Key for my admin user
2. Downloaded the AccessKeys.csv file
3. Configured the AWS CLI inside the instance using:

```bash
aws configure
```

I entered:
- Access Key ID
- Secret Access Key
- Region
- Default output (left blank)

After configuration, running:

```bash
aws s3 ls
```

successfully listed my S3 buckets.

---

## Creating and Accessing an S3 Bucket

I created a new S3 bucket:

```
nextwork-vpc-project-<yourname>
```

Uploaded two files via the console.

From the EC2 instance, I verified access using:

```bash
aws s3 ls s3://nextwork-vpc-project-<yourname>
```

This listed the objects stored inside the bucket.

<img width="507" height="310" alt="Screenshot 2026-02-15 at 4 04 16 PM" src="https://github.com/user-attachments/assets/2803d7d6-d717-45e5-961d-59460f2f16f3" />

---

## Uploading a File via CLI

I then tested uploading directly from EC2:

```bash
sudo touch /tmp/test.txt
aws s3 cp /tmp/test.txt s3://nextwork-vpc-project-<yourname>
aws s3 ls s3://nextwork-vpc-project-<yourname>
```

The final command confirmed that `test.txt` was successfully uploaded.

<img width="516" height="255" alt="Screenshot 2026-02-15 at 4 05 43 PM" src="https://github.com/user-attachments/assets/b2cd2e39-448c-4c28-b375-e2871796be0c" />
<img width="285" height="209" alt="Screenshot 2026-02-15 at 4 06 05 PM" src="https://github.com/user-attachments/assets/0a8f27c6-6e57-4556-8b71-e98f5d893537" />

---

## Security Consideration

In this project, I used IAM Access Keys for authentication.

However, best practice in production environments is to:

- Create an IAM Role
- Attach the role to the EC2 instance
- Avoid storing access keys on the server

This reduces long-term credential exposure and improves security posture.

---

## Phase 8 Outcome

- EC2 instance successfully authenticated with AWS
- S3 buckets accessed via AWS CLI
- Objects listed and uploaded from within the VPC
- Demonstrated how AWS services outside a VPC can be accessed securely

This phase bridges networking and IAM concepts, reinforcing how compute resources interact with managed AWS services.
