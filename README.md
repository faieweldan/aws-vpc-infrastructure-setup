# AWS VPC Architecture Project

A multi-phase deep dive into AWS networking, compute, and connectivity patterns.

## Project Roadmap

- [x] Phase 1: Build a Virtual Private Cloud  
- [x] Phase 2: VPC Traffic Flow and Security  
- [x] Phase 3: Creating a Private Subnet  
- [x] Phase 4: Launching VPC Resources  
- [x] Phase 5: Testing VPC Connectivity  
- [x] Phase 6: VPC Peering  
- [ ] Phase 7: VPC Monitoring with Flow Logs  
- [ ] Phase 8: Access S3 from a VPC  
- [ ] Phase 9: VPC Endpoints  


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

## Part 4: Launching VPC Resources

In this phase, I deployed compute resources inside both the public and private subnets to validate that my VPC architecture works as designed.

---

### Public EC2 Instance Deployment

I launched an EC2 instance inside the public subnet to confirm internet connectivity and routing behavior.

Configuration:

- Name: NextWork Public Server
- AMI: Amazon Linux 2023
- Instance Type: t2.micro
- VPC: NextWork VPC
- Subnet: Public Subnet
- Auto-assign Public IPv4: Enabled
- Security Group: NextWork Public Security Group
- Key Pair: RSA (.pem)

<img width="758" height="465" alt="image" src="https://github.com/user-attachments/assets/57d099bf-cc97-46d3-9d57-2cc98310af02" />


Key validation points:

- Instance received a Public IPv4 address
- Subnet routing correctly directed traffic to the Internet Gateway
- Security group allowed inbound HTTP (Port 80) from 0.0.0.0/0
- SSH access configured via key pair

This confirmed that the public subnet is properly exposed to the internet through routing and security rules.

---

### Private EC2 Instance Deployment

Next, I launched an EC2 instance inside the private subnet to validate network isolation.

Configuration:

- Name: NextWork Private Server
- AMI: Amazon Linux 2023
- Instance Type: t2.micro
- VPC: NextWork VPC
- Subnet: Private Subnet
- Public IPv4: Disabled
- Security Group: NextWork Private Security Group
- Key Pair: Same RSA key pair

Security configuration:

- Inbound SSH (Port 22) allowed only from NextWork Public Security Group
- No inbound access from 0.0.0.0/0
- No route to Internet Gateway in private route table

This ensures:

- Private instance is not publicly accessible
- Access is restricted to trusted internal resources
- Network isolation between public and private layers is enforced

<img width="756" height="464" alt="Screenshot 2026-02-12 at 6 40 42 PM" src="https://github.com/user-attachments/assets/4cc8fabe-21f4-46aa-85cb-f9f0b4b7799e" />


### VPC Wizard & Resource Map

To improve efficiency and understand AWS automation, I also used the “VPC and more” wizard:

- Generated public and private subnets automatically
- Observed route table and gateway creation
- Analyzed architecture using the VPC Resource Map
- Compared manual vs automated provisioning

This reinforced:

- High availability design across Availability Zones
- Public vs private subnet segmentation
- Route table separation
- NAT Gateway and endpoint planning (not deployed in this phase)

<img width="757" height="467" alt="Screenshot 2026-02-12 at 6 41 15 PM" src="https://github.com/user-attachments/assets/00c1aa8b-773f-4a65-a625-827e83f6b24c" />


### Phase Outcome

By the end of this phase:

- Public subnet supports internet-facing compute
- Private subnet remains isolated
- Security groups enforce controlled access
- Routing behaves exactly as intended

This completes the deployment validation of the VPC architecture.
<img width="675" height="581" alt="Screenshot 2026-02-12 at 6 41 46 PM" src="https://github.com/user-attachments/assets/7b16036b-8a86-4005-91b4-e13dc39a7828" />


## Part 5: Testing VPC Connectivity

In this phase, I validated network behavior across my VPC architecture by testing:

- SSH access to the public EC2 instance  
- Communication between public and private instances  
- Internet connectivity from within the VPC  

---

### Public EC2 Access (SSH)

I attempted to connect using EC2 Instance Connect.

Initial failure was caused by a missing SSH (Port 22) inbound rule in the Public Security Group.

After adding:
- SSH (Port 22)
- Source: 0.0.0.0/0 (temporary for testing)

Connection succeeded.

<img width="759" height="464" alt="Screenshot 2026-02-13 at 7 10 12 PM" src="https://github.com/user-attachments/assets/d91dab1e-126d-4033-8a6a-8c7c5d44ee6d" />

This confirmed correct Internet Gateway routing and security group configuration.

---

### Public → Private Instance Connectivity

From the public server, I tested:

```bash
ping <Private-IPv4>
```
Initial ping failed due to:

- Private Network ACL denying ICMP
- Private Security Group not allowing ICMP

Fix:
- Added ICMP rules in Private NACL (inbound + outbound)
- Added ICMP inbound rule in Private Security Group (source: Public SG)

Ping then succeeded.

<img width="782" height="369" alt="Screenshot 2026-02-13 at 7 11 05 PM" src="https://github.com/user-attachments/assets/1725b535-4134-426f-b851-63eecf0cb4d8" />
<img width="762" height="467" alt="Screenshot 2026-02-13 at 7 11 53 PM" src="https://github.com/user-attachments/assets/fe4b4ee7-ec07-46d7-8ba5-f7b1f4cc8919" />

This validated subnet-level (NACL) and instance-level (SG) filtering.

---

### Internet Connectivity Test

From the public server:

```bash
curl example.com
```

Returned HTML successfully.
<img width="757" height="466" alt="Screenshot 2026-02-13 at 7 12 12 PM" src="https://github.com/user-attachments/assets/5b983f08-8e5a-4cac-9968-65bea6dcbd30" />


This confirmed:
- Route table forwards 0.0.0.0/0 to Internet Gateway
- Outbound internet access is functioning

---

### Phase 5 Outcome

- Public instance is externally accessible via SSH  
- Private instance is isolated but reachable internally  
- Internet access from public subnet works  
- Security layers behave as designed  

The VPC architecture is now fully validated for connectivity.


## Part 6: VPC Peering

In this phase, I expanded my network architecture by connecting two isolated VPCs using VPC Peering and validating private cross-VPC communication.

---

### Multi-VPC Architecture

I created two separate VPCs:

- **VPC 1 CIDR:** 10.1.0.0/16  
- **VPC 2 CIDR:** 10.2.0.0/16  

The CIDR blocks must be unique to prevent routing conflicts and overlapping IP address ranges.

Each VPC contained:
- 1 public subnet
- 1 route table
- 1 default security group
- 1 EC2 instance

<img width="756" height="465" alt="Screenshot 2026-02-13 at 8 28 00 PM" src="https://github.com/user-attachments/assets/757bad7a-1f14-4625-b2af-1ec8b9f5287f" />

---

### Creating the Peering Connection

I created a VPC peering connection between:

- **Requester:** VPC 1  
- **Accepter:** VPC 2  

A VPC peering connection enables private IP communication between two VPCs without using the public internet.

<img width="757" height="469" alt="Screenshot 2026-02-13 at 8 28 23 PM" src="https://github.com/user-attachments/assets/f8ed8286-a81f-4c9b-9e6b-ad9a086f4288" />

---

### Updating Route Tables

Peering alone is not enough — traffic must know where to go.

I updated both route tables:

- VPC 1 route table:
  - Destination: `10.2.0.0/16`
  - Target: Peering Connection

- VPC 2 route table:
  - Destination: `10.1.0.0/16`
  - Target: Peering Connection

This allows bidirectional traffic between both networks.

<img width="760" height="464" alt="Screenshot 2026-02-13 at 8 28 46 PM" src="https://github.com/user-attachments/assets/54239feb-affe-46c4-a1c5-be68d70d14d8" />

---

### Launching EC2 Instances

I launched:

- 1 EC2 instance in VPC 1
- 1 EC2 instance in VPC 2

Key pairs were not configured manually since EC2 Instance Connect was used for access.

---

### Troubleshooting Instance Connect

Initial connection failed because:

- The instance had no public IP address.

To resolve this, I:
- Allocated an Elastic IP
- Associated it with the EC2 instance

This enabled SSH access through EC2 Instance Connect.

<img width="761" height="464" alt="Screenshot 2026-02-13 at 8 29 11 PM" src="https://github.com/user-attachments/assets/dd6e25a0-9320-451e-8413-a86642402fca" />
<img width="758" height="465" alt="Screenshot 2026-02-13 at 8 29 29 PM" src="https://github.com/user-attachments/assets/39ab7f4e-9207-4f75-b564-0d5953302811" />

---

### Testing VPC Peering

From Instance 1, I tested connectivity to Instance 2 using:

```bash
ping <Private-IP-of-VPC-2-Instance>
```

Initial ping failed because:

- VPC 2 Security Group did not allow ICMP from VPC 1.

Fix:
- Added inbound rule:
  - Type: All ICMP – IPv4
  - Source: 10.1.0.0/16

After updating the security group, ping replies succeeded.

<img width="758" height="465" alt="Screenshot 2026-02-13 at 8 29 44 PM" src="https://github.com/user-attachments/assets/e3ee62ac-b33f-495c-9d26-82e8eded11a9" />

---

### Phase 6 Outcome

- Two isolated VPCs were successfully peered
- Route tables were configured for cross-VPC routing
- Security groups were adjusted for controlled ICMP communication
- Private IP communication between VPCs was validated

This confirms a functional multi-VPC architecture using private networking.






AWS VPC Documentation: https : //docs.aws.amazon.com/vpc/
AWS CLI Command Reference : https://docs.aws.amazon.com/cli/latest/reference/ec2/
CIDR Block Calculator : https://www.ipaddressguide.com/cidr

