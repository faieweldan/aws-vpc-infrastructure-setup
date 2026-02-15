# Testing VPC Connectivity

In this phase, I validated network behavior across my VPC architecture by testing:

- SSH access to the public EC2 instance  
- Communication between public and private instances  
- Internet connectivity from within the VPC
  
<img width="548" height="578" alt="image" src="https://github.com/user-attachments/assets/ca06c5cf-35e1-4156-b367-518ef07d862f" />

---

## Public EC2 Access (SSH)

I attempted to connect using EC2 Instance Connect.

Initial failure was caused by a missing SSH (Port 22) inbound rule in the Public Security Group.

After adding:
- SSH (Port 22)
- Source: 0.0.0.0/0 (temporary for testing)

Connection succeeded.

<img width="759" height="464" alt="Screenshot 2026-02-13 at 7 10 12 PM" src="https://github.com/user-attachments/assets/d91dab1e-126d-4033-8a6a-8c7c5d44ee6d" />

This confirmed correct Internet Gateway routing and security group configuration.

---

## Public → Private Instance Connectivity

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

<img width="782" height="369" alt="Screenshot 2026-02-13 at 7 11 05 PM" src="https://github.com/user-attachments/assets/1725b535-4134-426f-b851-63eecf0cb4d8" />
<img width="762" height="467" alt="Screenshot 2026-02-13 at 7 11 53 PM" src="https://github.com/user-attachments/assets/fe4b4ee7-ec07-46d7-8ba5-f7b1f4cc8919" />

This validated subnet-level (NACL) and instance-level (SG) filtering.

---

## Internet Connectivity Test

From the public server:

```bash
curl example.com
```

Returned HTML successfully.
<img width="757" height="466" alt="Screenshot 2026-02-13 at 7 12 12 PM" src="https://github.com/user-attachments/assets/5b983f08-8e5a-4cac-9968-65bea6dcbd30" />


This confirmed:
- Route table forwards 0.0.0.0/0 to Internet Gateway
- Outbound internet access is functioning

---

## Phase 5 Outcome

- Public instance is externally accessible via SSH  
- Private instance is isolated but reachable internally  
- Internet access from public subnet works  
- Security layers behave as designed  

The VPC architecture is now fully validated for connectivity.
