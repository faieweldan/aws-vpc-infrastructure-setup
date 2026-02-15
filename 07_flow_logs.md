# VPC Monitoring with Flow Logs

In this phase, I extended my multi-VPC architecture by implementing **network-level monitoring using VPC Flow Logs and CloudWatch Logs Insights**.

This builds on the previous VPC Peering setup by adding visibility into network traffic.

<img width="546" height="571" alt="Screenshot 2026-02-15 at 4 07 50 PM" src="https://github.com/user-attachments/assets/9caf1d11-7163-4ee9-8cb0-d7d6be3109db" />

---

## Architecture Overview

- VPC 1 (10.1.0.0/16)
- VPC 2 (10.2.0.0/16)
- Public subnet in each VPC
- EC2 instance in each VPC
- VPC Peering connection established
- Route tables updated for private communication

<img width="758" height="464" alt="Screenshot 2026-02-14 at 9 52 50 PM" src="https://github.com/user-attachments/assets/b7738b27-9fc5-4a01-9671-5940c601597c" />

---

## New Focus: VPC Flow Logs

To monitor network traffic inside VPC 1, I configured:

- A CloudWatch Log Group
- A custom IAM Policy
- A dedicated IAM Role
- A VPC Flow Log attached to the VPC

Flow Logs were configured with:

- Filter: **ALL traffic**
- Aggregation interval: **1 minute**
- Destination: **CloudWatch Logs**

<img width="755" height="466" alt="Screenshot 2026-02-14 at 9 53 22 PM" src="https://github.com/user-attachments/assets/8a9ba644-f697-44e9-9dcf-b903aac383dd" />

---

## IAM Configuration for Flow Logs

Since VPC Flow Logs require permission to write to CloudWatch, I:

1. Created an IAM Policy allowing:
   - logs:CreateLogGroup
   - logs:CreateLogStream
   - logs:PutLogEvents
   - logs:DescribeLogGroups
   - logs:DescribeLogStreams

2. Created an IAM Role with a custom trust policy:
   - Principal: `vpc-flow-logs.amazonaws.com`

3. Attached the policy to the role
4. Assigned the role to the Flow Log

This allowed VPC Flow Logs to securely publish traffic logs to CloudWatch.

<img width="749" height="451" alt="Screenshot 2026-02-14 at 9 53 43 PM" src="https://github.com/user-attachments/assets/3925118c-a7d2-4d1d-a1c0-255d61236d76" />

---

## Generating Traffic for Monitoring

To generate traffic:

- Connected to EC2 in VPC 1 via EC2 Instance Connect
- Ran ping tests to EC2 in VPC 2 using:
  
```bash
ping <Private-IP-of-VPC-2>
```

- Verified connectivity after confirming:
  - Peering connection exists
  - Route tables include peering routes
  - Security groups allow ICMP

This produced measurable traffic for logging.

<img width="623" height="312" alt="Screenshot 2026-02-14 at 9 54 14 PM" src="https://github.com/user-attachments/assets/e4e266c5-dc37-45cc-a9dc-8114e525f899" />
<img width="440" height="396" alt="Screenshot 2026-02-14 at 9 55 03 PM" src="https://github.com/user-attachments/assets/d5189fd0-87db-44bb-b988-1cb1028d57f2" />

---

## Analyzing Traffic with CloudWatch Logs Insights

Using **CloudWatch Logs Insights**, I ran the query:

**Top 10 byte transfers by source and destination IP addresses**

This allowed me to:

- Identify highest data transfer pairs
- Detect ACCEPT vs REJECT traffic
- Observe EC2 Instance Connect traffic from public IP ranges
- Validate peering-based private communication

<img width="766" height="445" alt="Screenshot 2026-02-14 at 9 55 42 PM" src="https://github.com/user-attachments/assets/1cfaffef-de35-4886-a1d3-6efd4ba47905" />
<img width="767" height="443" alt="Screenshot 2026-02-14 at 9 55 58 PM" src="https://github.com/user-attachments/assets/9084ce83-fb4e-4b60-b7d4-7f1c2630daea" />

---

## Key Takeaways

- **VPC Flow Logs provide visibility into network-level traffic**
- IAM roles are required for service-to-service permissions
- Logs can be analyzed using CloudWatch Logs Insights queries
- Monitoring adds observability to previously built infrastructure
- Private VPC traffic can now be validated and audited

This phase transitioned the architecture from "connected" to **observable and monitorable**.
