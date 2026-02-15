# Private Subnet Architecture

To extend the VPC design beyond a basic public setup, I implemented a private subnet with isolated routing and stricter network controls.

This section focuses on understanding how internal resources are separated from direct internet exposure.

<img width="779" height="479" alt="image" src="https://github.com/user-attachments/assets/9dc6c116-bc4e-4b1c-bbfe-86283ff370da" />


## Private Subnet

A new subnet was created with a CIDR block that does not overlap with the public subnet.

Example:

* Public Subnet: `10.0.0.0/24`
* Private Subnet: `10.0.1.0/24`

The private subnet does not automatically have internet access. Internet connectivity depends entirely on routing configuration.

<img width="758" height="467" alt="Screenshot 2026-02-12 at 4 34 28 PM" src="https://github.com/user-attachments/assets/eb75638c-0e79-4511-bfe6-0ff7c7398b42" />


## Private Route Table

A dedicated route table was created specifically for the private subnet.

Key configuration:

* Contains only the local VPC route (`10.0.0.0/16 → local`)
* No route to an Internet Gateway
* No `0.0.0.0/0` internet route

This ensures that resources inside the private subnet cannot directly access or be accessed from the public internet.

The private subnet was explicitly associated with this route table.

<img width="757" height="465" alt="Screenshot 2026-02-12 at 4 34 52 PM" src="https://github.com/user-attachments/assets/11d3f825-15b4-42ef-a55a-cee6e8359d43" />


## Private Network ACL

A custom Network ACL was created and associated with the private subnet.

Key behavior:

* Custom NACLs deny all traffic by default
* Explicit inbound and outbound rules must be added to allow traffic

For this implementation:

* No broad public access rules were added
* Traffic is tightly controlled at the subnet boundary

This reinforces subnet-level isolation in addition to resource-level security groups.

<img width="759" height="467" alt="Screenshot 2026-02-12 at 4 35 16 PM" src="https://github.com/user-attachments/assets/41d07b9d-4cfc-486a-b364-7cc3d9f5f2fa" />

## Architecture Outcome

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


## Key Takeaways

* A subnet becomes private when it has no route to an Internet Gateway
* Route tables define exposure level
* Network ACLs enforce subnet-level traffic boundaries
* Isolation is achieved through routing + security layers, not naming conventions
