### Amazon VPC (Virtual Private Cloud)
**Definition:**
A logically isolated section of the AWS Cloud where you can launch AWS resources in a network that you  define.

**Key Features:**
- Choose your own IP address range (e.g., 10.0.0.0/16)
- Launch instances into subnets
- Configure route tables and gateways
- Supports IPv4 and IPv6
  Components:
- CIDR block
- Subnets (public/private)
- Route Tables
- Internet Gateway (IGW)
- NAT Gateway / NAT Instance
- VPC Peering, Transit Gateway

**Use Cases:**
- Host websites, databases, microservices in isolated environments
- Build hybrid cloud setups with on-premises connectivity

### Subnets

**Definition:**
A subnet is a range of IP addresses in your VPC.

**Types:**
- Public Subnet: Connected to the internet via an Internet Gateway
- Private Subnet: No direct route to the internet

**CIDR Blocks:**
- Define subnet size using subnet mask (e.g., /24 -> 256 IPs)

**Considerations:**
- Place resources in appropriate subnets based on access needs
- Can be `AZ-specific` for high availability


### Security Group
**Definition:**
A virtual firewall for EC2 instances to control inbound/outbound traffic.

**Key Points:**
- Stateful: Return traffic is automatically allowed
- Only allow rules (no deny rules)
- Rules are based on protocol, port, and source/destination

**Usage:**
- Secure applications by only allowing required ports (e.g., SSH 22, HTTP 80)
- Can be attached to multiple instances


## Routing (Route Tables)
**Definition:**
Route tables determine how traffic is directed within a VPC.

**Key Elements:**
- Destination: CIDR block of the traffic
- Target: Where to send the traffic (e.g., IGW, NAT Gateway, local, peering)

**Types:**
- Main Route Table: Associated with all subnets by default
- Custom Route Tables: Can be associated with specific subnets

**Special Routes:**
- 0.0.0.0/0 -> Internet Gateway (for public access)
- Local routes for VPC-internal communication

### Other Key Networking Components
1. Internet Gateway (IGW):
   - Enables internet access for instances in a VPC
   - Attach to the VPC and associate with public subnets
2. NAT Gateway:
   - Allows outbound internet traffic for private subnets
   - No inbound connections from the internet
3. VPC Peering:
   - Connects two VPCs for communication using private IPs
   - Cannot have transitive peering
4. Transit Gateway:
   - Central hub to connect multiple VPCs and on-prem networks
   - Scales better than VPC peering in large architectures