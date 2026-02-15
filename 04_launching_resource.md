# Launching VPC Resources

In this phase, I deployed compute resources inside both the public and private subnets to validate that my VPC architecture works as designed.

---

## Public EC2 Instance Deployment

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

## Private EC2 Instance Deployment

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

<img width="756" height="464" alt="Screenshot 2026-02-12 at 6 40 42 PM" src="https://github.com/user-attachments/assets/4cc8fabe-21f4-46aa-85cb-f9f0b4b7799e" />


## VPC Wizard & Resource Map

To improve efficiency and understand AWS automation, I also used the "VPC and more" wizard:

- Generated public and private subnets automatically
- Observed route table and gateway creation
- Analyzed architecture using the VPC Resource Map
- Compared manual vs automated provisioning

This reinforced:

- High availability design across Availability Zones
- Public vs private subnet segmentation
- Route table separation
- NAT Gateway and endpoint planning (not deployed in this phase)

<img width="757" height="467" alt="Screenshot 2026-02-12 at 6 41 15 PM" src="https://github.com/user-attachments/assets/00c1aa8b-773f-4a65-a625-827e83f6b24c" />


## Phase Outcome

By the end of this phase:

- Public subnet supports internet-facing compute
- Private subnet remains isolated
- Security groups enforce controlled access
- Routing behaves exactly as intended

This completes the deployment validation of the VPC architecture.
<img width="675" height="581" alt="Screenshot 2026-02-12 at 6 41 46 PM" src="https://github.com/user-attachments/assets/7b16036b-8a86-4005-91b4-e13dc39a7828" />
