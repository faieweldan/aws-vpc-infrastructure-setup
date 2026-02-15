# VPC Traffic Flow & Security Configuration

After setting up the base VPC, subnet, and internet gateway, the next step was configuring how traffic actually flows in and out of the VPC, and how that traffic is secured. This involved working with route tables, security groups, and network ACLs.

## Internet Gateway

An Internet Gateway was created and attached to the VPC. This provides a path between the VPC and the public internet. On its own, attaching an internet gateway does not make any resources public — it only enables the possibility of internet access.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/d921cab3-ba05-44db-a188-d4f1027b88ec" />


## Route Table (Public Subnet Routing)

A custom route table was created for the VPC to control how network traffic is directed.

* The route table initially contained only a local route for internal VPC traffic (`10.0.0.0/16`)
* A new route was added with:

  * **Destination:** `0.0.0.0/0`
  * **Target:** Internet Gateway

This default route sends all non-VPC traffic to the internet. After associating this route table with the public subnet, the subnet officially became a public subnet.

This step clarified that a subnet is considered "public" only when its route table points to an internet gateway.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/09c1ee06-e5ce-47f0-a99c-157378ef4bd2" />


## Security Group (Resource-Level Control)

A custom security group was created for resources inside the VPC.

* Inbound rule allows HTTP traffic (Port 80) from `0.0.0.0/0`
* Outbound traffic is allowed by default

Security groups operate at the resource level and are stateful, meaning return traffic is automatically allowed if the inbound request is permitted. This is typically used to control access to services like web servers or APIs.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/d7325557-8343-4a9c-8aa0-89210ee82402" />


## Network ACL (Subnet-Level Control)

To add another layer of security, a custom network ACL was created and associated with the public subnet.

* Custom network ACLs deny all traffic by default
* Explicit inbound and outbound rules were added:

  * **Inbound:** allow all traffic from `0.0.0.0/0`
  * **Outbound:** allow all traffic to `0.0.0.0/0`

Network ACLs apply to the entire subnet and are stateless, meaning both inbound and outbound rules must explicitly allow traffic. Only one network ACL can be associated with a subnet at a time.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/8cfef812-a89b-4b16-8a02-6d1c34fbcba5" />


## End-to-End Traffic Flow

With all components in place, internet traffic flows as follows:

* A user sends a request from the internet
* The request enters through the Internet Gateway
* The route table forwards the traffic to the public subnet
* The network ACL allows the packet into the subnet
* The security group allows the request to reach the resource
* The response follows the same path back to the user

Each component must allow the traffic for communication to succeed.

---

## Additional Practice

To reinforce these concepts, the same VPC components were also created using the AWS CLI through CloudShell, including testing resource creation in a different AWS region. EC2 Global View was used to verify and track resources across regions.
