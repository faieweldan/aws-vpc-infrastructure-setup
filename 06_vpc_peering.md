# VPC Peering

In this phase, I expanded my network architecture by connecting two isolated VPCs using VPC Peering and validating private cross-VPC communication.

<img width="548" height="584" alt="Screenshot 2026-02-15 at 4 08 29 PM" src="https://github.com/user-attachments/assets/860ed71e-f78d-4f2d-b144-2bfac4b2e05c" />

---

## Multi-VPC Architecture

I created two separate VPCs:

- **VPC 1 CIDR:** 10.1.0.0/16  
- **VPC 2 CIDR:** 10.2.0.0/16  

The CIDR blocks must be unique to prevent routing conflicts and overlapping IP address ranges.

Each VPC contained:
- 1 public subnet
- 1 route table
- 1 default security group
- 1 EC2 instance

<img width="756" height="465" alt="Screenshot 2026-02-13 at 8 28 00 PM" src="https://github.com/user-attachments/assets/757bad7a-1f14-4625-b2af-1ec8b9f5287f" />

---

## Creating the Peering Connection

I created a VPC peering connection between:

- **Requester:** VPC 1  
- **Accepter:** VPC 2  

A VPC peering connection enables private IP communication between two VPCs without using the public internet.

<img width="757" height="469" alt="Screenshot 2026-02-13 at 8 28 23 PM" src="https://github.com/user-attachments/assets/f8ed8286-a81f-4c9b-9e6b-ad9a086f4288" />

---

## Updating Route Tables

Peering alone is not enough — traffic must know where to go.

I updated both route tables:

- VPC 1 route table:
  - Destination: `10.2.0.0/16`
  - Target: Peering Connection

- VPC 2 route table:
  - Destination: `10.1.0.0/16`
  - Target: Peering Connection

This allows bidirectional traffic between both networks.

<img width="760" height="464" alt="Screenshot 2026-02-13 at 8 28 46 PM" src="https://github.com/user-attachments/assets/54239feb-affe-46c4-a1c5-be68d70d14d8" />

---

## Launching EC2 Instances

I launched:

- 1 EC2 instance in VPC 1
- 1 EC2 instance in VPC 2

Key pairs were not configured manually since EC2 Instance Connect was used for access.

---

## Troubleshooting Instance Connect

Initial connection failed because:

- The instance had no public IP address.

To resolve this, I:
- Allocated an Elastic IP
- Associated it with the EC2 instance

This enabled SSH access through EC2 Instance Connect.

<img width="761" height="464" alt="Screenshot 2026-02-13 at 8 29 11 PM" src="https://github.com/user-attachments/assets/dd6e25a0-9320-451e-8413-a86642402fca" />
<img width="758" height="465" alt="Screenshot 2026-02-13 at 8 29 29 PM" src="https://github.com/user-attachments/assets/39ab7f4e-9207-4f75-b564-0d5953302811" />

---

## Testing VPC Peering

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

<img width="758" height="465" alt="Screenshot 2026-02-13 at 8 29 44 PM" src="https://github.com/user-attachments/assets/e3ee62ac-b33f-495c-9d26-82e8eded11a9" />

---

## Phase 6 Outcome

- Two isolated VPCs were successfully peered
- Route tables were configured for cross-VPC routing
- Security groups were adjusted for controlled ICMP communication
- Private IP communication between VPCs was validated

This confirms a functional multi-VPC architecture using private networking.
