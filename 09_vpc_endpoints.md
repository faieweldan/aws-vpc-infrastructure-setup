# VPC Endpoints (Private S3 Routing)

> Continuing from Part 8, where EC2 accessed S3 using IAM access keys over the public internet.

In this phase, I improved the architecture by implementing a **Gateway VPC Endpoint** for Amazon S3 to eliminate public internet routing and enforce private access.

---

## Why This Was Necessary

In Part 8:
- EC2 accessed S3 successfully
- Traffic flowed through the Internet Gateway
- Communication left the VPC and traversed the public AWS network

This works — but it is not ideal for secure production workloads.

Goal of Part 9:
Redirect S3 traffic internally using a VPC Endpoint and restrict bucket access accordingly.
<img width="546" height="308" alt="Screenshot 2026-02-15 at 5 32 47 PM" src="https://github.com/user-attachments/assets/c07977df-5f6c-4e74-a6c8-cc2a472ec1cb" />

---

## Step 1 – Create S3 Gateway Endpoint

Actions performed:

1. VPC → Endpoints → Create Endpoint
2. Service: `com.amazonaws.<region>.s3`
3. Type: **Gateway**
4. Attached to `NextWork-vpc`
5. Associated with Public Route Table

After creation, AWS added a new route:

Destination:
```
pl-xxxx (S3 Prefix List)
```

Target:
```
vpce-xxxx
```

This ensures S3-bound traffic no longer uses the Internet Gateway.

<img width="639" height="371" alt="Screenshot 2026-02-15 at 5 33 28 PM" src="https://github.com/user-attachments/assets/9a85ad97-cebe-4c4e-b44d-6fe80b67675a" />

---

## Step 2 – Enforce Bucket Restriction

To validate private routing, I applied a restrictive bucket policy:

- Deny all S3 actions
- Allow access only if request originates from the VPC endpoint

Policy condition used:

```json
"Condition": {
  "StringNotEquals": {
    "aws:sourceVpce": "vpce-xxxxxxxx"
  }
}
```

After applying:

- Console access was denied
- Public internet access was blocked
- Only endpoint traffic permitted

<img width="639" height="370" alt="Screenshot 2026-02-15 at 5 34 02 PM" src="https://github.com/user-attachments/assets/171d5bf0-9e35-4180-a704-8857ed351298" />
<img width="641" height="372" alt="Screenshot 2026-02-15 at 5 34 21 PM" src="https://github.com/user-attachments/assets/38196390-d06f-48a1-a245-ead651d01f41" />

---

## Step 3 – Troubleshooting Access Denied

Initial endpoint test failed.

Root cause:
- Route table not properly associated with endpoint

Fix:
- Managed route table associations
- Verified S3 prefix list route existed in subnet route table

Retested:

```bash
aws s3 ls s3://nextwork-vpc-project-<yourname>
```

Access succeeded via endpoint.

<img width="640" height="370" alt="Screenshot 2026-02-15 at 5 35 23 PM" src="https://github.com/user-attachments/assets/92966482-cc49-4bea-b111-3093caac6e05" />
<img width="644" height="369" alt="Screenshot 2026-02-15 at 5 35 46 PM" src="https://github.com/user-attachments/assets/d7bb2228-d6ff-4898-8f73-0dc47b0a5858" />

---

## Step 4 – Endpoint Policy Validation

To test control layer effectiveness:

Changed endpoint policy from:

```json
"Effect": "Allow"
```

To:

```json
"Effect": "Deny"
```

Result:
- EC2 immediately lost S3 access

Reverted policy back to Allow for cleanup.

This demonstrated:
- Endpoint policies act as an additional control layer
- Network + IAM + bucket policy combine for layered security

---

## Final Architecture Improvement

Part 8:
EC2 → Internet Gateway → S3

Part 9:
EC2 → Route Table → S3 Gateway Endpoint → S3

✔ No public internet traversal  
✔ Bucket locked to endpoint traffic only  
✔ Verified route-based redirection  
✔ Validated endpoint policy enforcement  

This completes the networking series by implementing private AWS service access using VPC endpoints.
