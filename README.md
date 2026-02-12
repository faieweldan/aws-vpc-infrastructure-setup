# AWS VPC Infrastructure Setup

A hands-on project demonstrating how to build a secure AWS Virtual Private Cloud (VPC) with public/private subnet architecture, internet gateway configuration, and custom IP addressing using both AWS Console and CLI.

Project Overview

Built a custom VPC infrastructure from scratch to understand AWS networking fundamentals. This project covers creating isolated network environments, configuring subnets across availability zones, and establishing internet connectivity for cloud resources.

Region: US East (N. Virginia) - us-east-1

What I Built

- VPC with custom CIDR block (10.0.0.0/16)
- Public Subnet in us-east-1b (10.0.0.0/24)
- Internet Gateway attached to VPC
- Auto-assign public IPv4 enabled for subnet

Skills and Technologies

- AWS Command Line Interface (CLI)
- AWS CloudShell
- AWS Identity and Access Management (IAM)
- Amazon VPC
- CIDR notation and IP addressing

Documentation

- Console Method Guide - Step-by-step using AWS web interface
- CLI Method Guide - Automated setup using AWS CLI commands

Quick Start (CLI Method)

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text

# Create subnet
aws ec2 create-subnet --vpc-id <VPC-ID> --cidr-block 10.0.0.0/24

# Create and attach internet gateway
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
aws ec2 attach-internet-gateway --vpc-id <VPC-ID> --internet-gateway-id <IGW-ID>
```

Key Learnings

VPCs as Isolated Networks: VPCs provide logical isolation in AWS, similar to having your own private data center. Without VPCs, all resources would exist in one shared space with no security boundaries.

Public vs Private Subnets: A subnet becomes "public" only when connected to an internet gateway. The naming is just convention - actual internet connectivity requires proper gateway attachment.

CIDR Block Planning: Used /16 for VPC (65,536 IPs) and /24 for subnet (256 IPs). Smaller number after slash = larger address space.

CLI vs Console Trade-offs: 
- CLI is faster for repetitive tasks and automation
- Console provides better visualization and is more beginner-friendly
- CLI requires exact syntax but enables infrastructure-as-code


📸 Screenshots
<img width="784" height="482" alt="Screenshot 2026-01-28 at 12 03 31 AM" src="https://github.com/user-attachments/assets/39311996-727c-49dd-a9f6-58fe139efa2e" />
<img width="753" height="466" alt="Screenshot 2026-01-28 at 12 03 57 AM" src="https://github.com/user-attachments/assets/6a2e6746-2342-4207-b27d-5d0a2c42e2c3" />
<img width="752" height="468" alt="Screenshot 2026-01-28 at 12 04 08 AM" src="https://github.com/user-attachments/assets/281f83a3-9709-4bb0-a72a-5401793437e5" />
<img width="460" height="285" alt="image" src="https://github.com/user-attachments/assets/5c41a8a3-1b57-4574-a169-66776f700cef" />
<img width="1437" height="691" alt="Screenshot 2026-01-27 at 11 51 53 PM" src="https://github.com/user-attachments/assets/b4eb6bc5-b95e-4bd7-9a11-3359bcc23540" />


Cleanup

To avoid AWS charges, delete resources in this order:

```bash
aws ec2 detach-internet-gateway --vpc-id <VPC-ID> --internet-gateway-id <IGW-ID>
aws ec2 delete-internet-gateway --internet-gateway-id <IGW-ID>
aws ec2 delete-subnet --subnet-id <SUBNET-ID>
aws ec2 delete-vpc --vpc-id <VPC-ID>
```

## VPC Traffic Flow & Security Configuration

After setting up the base VPC, subnet, and internet gateway, the next step was configuring how traffic actually flows in and out of the VPC, and how that traffic is secured. This involved working with route tables, security groups, and network ACLs.

### Internet Gateway

An Internet Gateway was created and attached to the VPC. This provides a path between the VPC and the public internet. On its own, attaching an internet gateway does not make any resources public — it only enables the possibility of internet access.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/d921cab3-ba05-44db-a188-d4f1027b88ec" />


### Route Table (Public Subnet Routing)

A custom route table was created for the VPC to control how network traffic is directed.

* The route table initially contained only a local route for internal VPC traffic (`10.0.0.0/16`)
* A new route was added with:

  * **Destination:** `0.0.0.0/0`
  * **Target:** Internet Gateway

This default route sends all non-VPC traffic to the internet. After associating this route table with the public subnet, the subnet officially became a public subnet.

This step clarified that a subnet is considered “public” only when its route table points to an internet gateway.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/09c1ee06-e5ce-47f0-a99c-157378ef4bd2" />


### Security Group (Resource-Level Control)

A custom security group was created for resources inside the VPC.

* Inbound rule allows HTTP traffic (Port 80) from `0.0.0.0/0`
* Outbound traffic is allowed by default

Security groups operate at the resource level and are stateful, meaning return traffic is automatically allowed if the inbound request is permitted. This is typically used to control access to services like web servers or APIs.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/d7325557-8343-4a9c-8aa0-89210ee82402" />


### Network ACL (Subnet-Level Control)

To add another layer of security, a custom network ACL was created and associated with the public subnet.

* Custom network ACLs deny all traffic by default
* Explicit inbound and outbound rules were added:

  * **Inbound:** allow all traffic from `0.0.0.0/0`
  * **Outbound:** allow all traffic to `0.0.0.0/0`

Network ACLs apply to the entire subnet and are stateless, meaning both inbound and outbound rules must explicitly allow traffic. Only one network ACL can be associated with a subnet at a time.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/8cfef812-a89b-4b16-8a02-6d1c34fbcba5" />


### End-to-End Traffic Flow

With all components in place, internet traffic flows as follows:

* A user sends a request from the internet
* The request enters through the Internet Gateway
* The route table forwards the traffic to the public subnet
* The network ACL allows the packet into the subnet
* The security group allows the request to reach the resource
* The response follows the same path back to the user

Each component must allow the traffic for communication to succeed.

---

### Additional Practice

To reinforce these concepts, the same VPC components were also created using the AWS CLI through CloudShell, including testing resource creation in a different AWS region. EC2 Global View was used to verify and track resources across regions.

Perfect 👍 I’ll continue from the exact last line of the previous README section so you know exactly where to paste this.


## Part 3: Private Subnet Architecture

To extend the VPC design beyond a basic public setup, I implemented a private subnet with isolated routing and stricter network controls.

This section focuses on understanding how internal resources are separated from direct internet exposure.

<img width="779" height="479" alt="image" src="https://github.com/user-attachments/assets/9dc6c116-bc4e-4b1c-bbfe-86283ff370da" />


### Private Subnet

A new subnet was created with a CIDR block that does not overlap with the public subnet.

Example:

* Public Subnet: `10.0.0.0/24`
* Private Subnet: `10.0.1.0/24`

The private subnet does not automatically have internet access. Internet connectivity depends entirely on routing configuration.

<img width="758" height="467" alt="Screenshot 2026-02-12 at 4 34 28 PM" src="https://github.com/user-attachments/assets/eb75638c-0e79-4511-bfe6-0ff7c7398b42" />


### Private Route Table

A dedicated route table was created specifically for the private subnet.

Key configuration:

* Contains only the local VPC route (`10.0.0.0/16 → local`)
* No route to an Internet Gateway
* No `0.0.0.0/0` internet route

This ensures that resources inside the private subnet cannot directly access or be accessed from the public internet.

The private subnet was explicitly associated with this route table.

<img width="757" height="465" alt="Screenshot 2026-02-12 at 4 34 52 PM" src="https://github.com/user-attachments/assets/11d3f825-15b4-42ef-a55a-cee6e8359d43" />


### Private Network ACL

A custom Network ACL was created and associated with the private subnet.

Key behavior:

* Custom NACLs deny all traffic by default
* Explicit inbound and outbound rules must be added to allow traffic

For this implementation:

* No broad public access rules were added
* Traffic is tightly controlled at the subnet boundary

This reinforces subnet-level isolation in addition to resource-level security groups.

<img width="759" height="467" alt="Screenshot 2026-02-12 at 4 35 16 PM" src="https://github.com/user-attachments/assets/41d07b9d-4cfc-486a-b364-7cc3d9f5f2fa" />

### Architecture Outcome

The VPC now consists of:

* Public Subnet

  * Has route to Internet Gateway
  * Designed for internet-facing resources

* Private Subnet

  * No route to Internet Gateway
  * Designed for internal services or backend systems

This structure reflects a common production-ready pattern where:

* Public subnets host entry-point resources
* Private subnets host internal components
* Routing and security layers control traffic flow between them

<img width="615" height="538" alt="image" src="https://github.com/user-attachments/assets/47d8f68a-44a0-4e4b-af96-4f932e9ac639" />


### Key Takeaways

* A subnet becomes private when it has no route to an Internet Gateway
* Route tables define exposure level
* Network ACLs enforce subnet-level traffic boundaries
* Isolation is achieved through routing + security layers, not naming conventions




AWS VPC Documentation: https : //docs.aws.amazon.com/vpc/
AWS CLI Command Reference : https://docs.aws.amazon.com/cli/latest/reference/ec2/
CIDR Block Calculator : https://www.ipaddressguide.com/cidr

