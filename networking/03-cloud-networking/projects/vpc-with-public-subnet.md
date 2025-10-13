### Create a Public Subnet and EC2 inside a VPC

#### Create a VPC:
- Name: nginx-test.
- CIDR Block: 10.0.0.0/16 (provides 65,536 IP addresses).
- Purpose: This VPC will contain all network resources, including subnets and the EC2 instance.

##### Create a Public Subnet:
- Name: nginx-test-public.
- VPC: Select nginx-test.
- CIDR Block: 10.0.1.0/24 (provides 256 IP addresses, 251 usable).
- Availability Zone: Choose one (e.g., us-east-1a).
- Purpose: This subnet will host resources (e.g., EC2 instance) that need public internet access.

#### Create and Attach an Internet Gateway:
- Name: nginx-test-igw.
- Action: Create an Internet Gateway and attach it to the nginx-test VPC.
- Purpose: The Internet Gateway enables communication between the VPC and the public internet.

#### Create and Configure a Route Table:
- Name: nginx-test-public-rt.
- VPC: Select nginx-test.
- Add Route:
  - Destination: 0.0.0.0/0 (all external traffic).
  - Target: Select the Internet Gateway (nginx-test-igw).
- Purpose: This route table directs traffic from the subnet to the internet via the Internet Gateway.
  
#### Associate the Public Subnet with the Route Table:
- Action: Associate the nginx-test-public subnet with the nginx-test-public-rt route table.
-  Note: A subnet can only be associated with one route table at a time.
- Purpose: This ensures the subnet has internet access, making it a public subnet.

#### Launch an EC2 Instance in the Public Subnet:
- AMI: Choose Ubuntu Server 22.04 LTS (or latest).
- Instance Type: t2.micro (free tier eligible).
- VPC: Select nginx-test.
- Subnet: Select nginx-test-public.
- Auto-assign Public IP: Enable (ensures the instance gets a public IP for internet access).
- Security Group: Allow SSH (port 22) and HTTP (port 80) from 0.0.0.0/0 (or restrict to your IP for security).
- Key Pair: Create or select a key pair for SSH access.
- Purpose: The EC2 instance will be publicly accessible and can run a web server like NGINX.