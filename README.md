AWS VPC Infrastructure Setup
A hands-on project demonstrating how to build a secure AWS Virtual Private Cloud (VPC) with public/private subnet architecture, internet gateway configuration, and custom IP addressing using both AWS Console and CLI.
🎯 Project Overview
Built a custom VPC infrastructure from scratch to understand AWS networking fundamentals. This project covers creating isolated network environments, configuring subnets across availability zones, and establishing internet connectivity for cloud resources.
Timeline: January 2026
Region: US East (N. Virginia) - us-east-1
🏗️ What I Built

VPC with custom CIDR block (10.0.0.0/16)
Public Subnet in us-east-1b (10.0.0.0/24)
Internet Gateway attached to VPC
Auto-assign public IPv4 enabled for subnet

🛠️ Skills & Technologies

AWS Command Line Interface (CLI)
AWS CloudShell
AWS Identity and Access Management (IAM)
Amazon VPC
CIDR notation and IP addressing

📚 Documentation

Console Method Guide - Step-by-step using AWS web interface
CLI Method Guide - Automated setup using AWS CLI commands
Troubleshooting - Common errors and solutions

🚀 Quick Start (CLI Method)
bash# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text

# Create subnet
aws ec2 create-subnet --vpc-id <VPC-ID> --cidr-block 10.0.0.0/24

# Create and attach internet gateway
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
aws ec2 attach-internet-gateway --vpc-id <VPC-ID> --internet-gateway-id <IGW-ID>
See scripts/ folder for complete commands.
💡 Key Learnings
VPCs as Isolated Networks: VPCs provide logical isolation in AWS, similar to having your own private data center. Without VPCs, all resources would exist in one shared space with no security boundaries.
Public vs Private Subnets: A subnet becomes "public" only when connected to an internet gateway. The naming is just convention - actual internet connectivity requires proper gateway attachment.
CIDR Block Planning: Used /16 for VPC (65,536 IPs) and /24 for subnet (256 IPs). Smaller number after slash = larger address space.
CLI vs Console Trade-offs:

CLI is faster for repetitive tasks and automation
Console provides better visualization and is more beginner-friendly
CLI requires exact syntax but enables infrastructure-as-code

📸 Screenshots
<img width="784" height="482" alt="Screenshot 2026-01-28 at 12 03 31 AM" src="https://github.com/user-attachments/assets/39311996-727c-49dd-a9f6-58fe139efa2e" />
<img width="753" height="466" alt="Screenshot 2026-01-28 at 12 03 57 AM" src="https://github.com/user-attachments/assets/6a2e6746-2342-4207-b27d-5d0a2c42e2c3" />
<img width="752" height="468" alt="Screenshot 2026-01-28 at 12 04 08 AM" src="https://github.com/user-attachments/assets/281f83a3-9709-4bb0-a72a-5401793437e5" />
<img width="460" height="285" alt="image" src="https://github.com/user-attachments/assets/5c41a8a3-1b57-4574-a169-66776f700cef" />
<img width="1437" height="691" alt="Screenshot 2026-01-27 at 11 51 53 PM" src="https://github.com/user-attachments/assets/b4eb6bc5-b95e-4bd7-9a11-3359bcc23540" />




🔄 Cleanup
To avoid AWS charges, delete resources in this order:
bashaws ec2 detach-internet-gateway --vpc-id <VPC-ID> --internet-gateway-id <IGW-ID>
aws ec2 delete-internet-gateway --internet-gateway-id <IGW-ID>
aws ec2 delete-subnet --subnet-id <SUBNET-ID>
aws ec2 delete-vpc --vpc-id <VPC-ID>

Preferred Method: Console for learning and visualization, but I can see CLI becoming more efficient for real projects with multiple environments.

📝 Notes
The hands-on practice with both methods helped solidify understanding of how VPC components interact. 

AWS VPC Documentation: https : //docs.aws.amazon.com/vpc/
AWS CLI Command Reference : https://docs.aws.amazon.com/cli/latest/reference/ec2/
CIDR Block Calculator : https://www.ipaddressguide.com/cidr

