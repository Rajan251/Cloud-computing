# Complete AWS VPC Guide: Components, Configuration & Implementation

## Table of Contents
1. [VPC Overview](#vpc-overview)
2. [VPC Core Components](#vpc-core-components)
3. [VPC Networking Components](#vpc-networking-components)
4. [VPC Security Components](#vpc-security-components)
5. [VPC Connectivity Components](#vpc-connectivity-components)
6. [Step-by-Step VPC Configuration](#step-by-step-vpc-configuration)
7. [Advanced VPC Features](#advanced-vpc-features)
8. [Best Practices](#best-practices)

## VPC Overview

Amazon Virtual Private Cloud (VPC) is a logically isolated virtual network within the AWS cloud where you can launch AWS resources. It provides complete control over your virtual networking environment, including IP address ranges, subnets, route tables, and network gateways.

### Key Benefits:
- **Isolation**: Logical separation from other virtual networks
- **Control**: Full control over network configuration
- **Security**: Multiple layers of security controls
- **Flexibility**: Customizable network topology
- **Integration**: Seamless integration with AWS services

## VPC Core Components

### 1. VPC (Virtual Private Cloud)
The foundational component that defines your isolated network environment.

**Key Characteristics:**
- Regional resource (spans multiple Availability Zones)
- Defined by CIDR block (IP address range)
- Default limit: 5 VPCs per region (can be increased via support request)
- Cannot be changed once created (CIDR can be extended)
- IPv6 CIDR block support (Optional)
- Dedicated tenancy option (Optional - for compliance requirements)

### 2. Subnets
Subdivisions of your VPC's IP address range, located in specific Availability Zones.

**Types:**
- **Public Subnet**: Has route to Internet Gateway
- **Private Subnet**: No direct route to internet
- **Isolated Subnet**: No internet access, used for databases
- **Outpost Subnet** (Optional - for AWS Outposts)

**Key Points:**
- AZ-specific (cannot span multiple AZs)
- CIDR block must be subset of VPC CIDR
- First 4 and last IP addresses are reserved by AWS
- Auto-assign public IPv4 (Optional - configurable per subnet)
- IPv6 support (Optional)

### 3. Availability Zones (AZs)
Physically separate data centers within a region where you can place subnets.

**Benefits:**
- High availability and fault tolerance
- Low latency between AZs in same region
- Independent infrastructure
- Local Zones (Optional - for ultra-low latency applications)
- Wavelength Zones (Optional - for 5G edge computing)

## VPC Networking Components

### 4. Internet Gateway (IGW)
Allows communication between instances in your VPC and the internet.

**Characteristics:**
- One per VPC (required for internet access)
- Horizontally scaled, redundant, and highly available
- No bandwidth constraints
- Performs NAT for instances with public IP addresses
- IPv6 support (Optional)
- Egress-only Internet Gateway (Optional - for IPv6 outbound-only traffic)

### 5. NAT Gateway
Enables instances in private subnets to connect to the internet for outbound traffic.

**Features:**
- Managed service (no maintenance required)
- AZ-specific (create one per AZ for high availability)
- Scales automatically up to 45 Gbps
- Charges apply for usage and data processing
- Bandwidth scaling (Optional - can request higher limits)
- IPv6 support (Optional)

### 6. NAT Instance (Optional/Legacy)
EC2 instance configured to provide NAT functionality (legacy approach).

**When to Use:**
- Need more control over NAT configuration
- Cost optimization for small workloads
- Custom security requirements
- Port forwarding requirements (Optional)
- Custom routing or filtering (Optional)

### 7. Route Tables
Define routing rules for network traffic within your VPC.

**Types:**
- **Main Route Table**: Default for all subnets
- **Custom Route Tables**: Associated with specific subnets
- **Gateway Route Tables** (Optional - for routing through virtual appliances)

**Key Concepts:**
- Local routes (within VPC) are automatically added
- Most specific route takes precedence
- Each subnet must be associated with a route table
- Route propagation (Optional - for VPN/Direct Connect)
- Edge associations (Optional - for Internet/NAT Gateways)

### 8. Elastic Network Interface (ENI)
Virtual network interface that can be attached to instances.

**Features:**
- Primary and secondary private IP addresses
- Elastic IP addresses (Optional)
- MAC address
- Security groups
- Source/destination check flag
- IPv6 addresses (Optional)
- SR-IOV support (Optional - for enhanced networking)

## VPC Security Components

### 9. Security Groups
Virtual firewalls that control inbound and outbound traffic at the instance level.

**Characteristics:**
- **Stateful**: Return traffic is automatically allowed
- **Default Deny**: All inbound traffic denied by default
- **Allow Rules Only**: Cannot create deny rules
- **Instance Level**: Applied to ENIs
- Cross-security group references (Optional - for complex architectures)
- IPv6 rules support (Optional)
- Rule descriptions (Optional - for documentation)

### 10. Network Access Control Lists (NACLs)
Subnet-level firewalls that control traffic entering and leaving subnets.

**Characteristics:**
- **Stateless**: Must explicitly allow return traffic
- **Numbered Rules**: Processed in order (lowest number first)
- **Allow and Deny Rules**: Can create both types
- **Subnet Level**: Applied to all instances in subnet
- Default NACL (allows all traffic)
- IPv6 rules support (Optional)
- Custom NACLs (Optional - for enhanced security)

### 11. VPC Flow Logs
Capture information about IP traffic going to and from network interfaces.

**Levels:**
- VPC level
- Subnet level
- Network interface level
- Transit Gateway level (Optional)

**Destinations:**
- CloudWatch Logs
- S3
- Kinesis Data Firehose
- Custom format (Optional - specify which fields to capture)
- Parquet format (Optional - for cost optimization in S3)

## VPC Connectivity Components

### 12. VPC Peering
Network connection between two VPCs for private communication.

**Characteristics:**
- Cross-region and cross-account support
- Non-transitive (must create direct connections)
- No single point of failure
- CIDR blocks cannot overlap
- DNS resolution support (Optional - enable DNS hostname resolution)
- Jumbo frames support (Optional - up to 9000 MTU)
- Inter-region peering encryption (Optional - automatic encryption)

### 13. VPC Endpoints
Enable private connectivity to AWS services without internet gateway.

**Types:**
- **Gateway Endpoints**: S3 and DynamoDB (free)
- **Interface Endpoints**: Most AWS services (powered by PrivateLink)
- **Gateway Load Balancer Endpoints** (Optional - for third-party appliances)

**Features:**
- Private DNS names (Optional - for interface endpoints)
- Custom DNS names (Optional)
- Cross-region support (Optional - for some services)
- Endpoint policies (Optional - for access control)

### 14. AWS Transit Gateway (Optional)
Central hub for connecting VPCs and on-premises networks.

**Benefits:**
- Simplifies network topology
- Scales to thousands of connections
- Centralized routing control
- Cross-region support
- Direct Connect Gateway integration (Optional)
- VPN attachment support (Optional)
- Multicast support (Optional)
- Network Manager integration (Optional - for monitoring)

### 15. Direct Connect (Optional)
Dedicated network connection from on-premises to AWS.

**Benefits:**
- Consistent network performance
- Reduced bandwidth costs
- Private connectivity to VPCs
- Virtual interfaces (VIFs) support
- BGP routing
- Dedicated/Hosted connections (Optional)
- Link Aggregation Groups (LAG) (Optional)
- MACsec encryption (Optional)

### 16. VPN Connections (Optional)
Encrypted connections between your VPC and on-premises network.

**Types:**
- **Site-to-Site VPN**: IPsec tunnel
- **Client VPN**: Remote access for users
- **AWS VPN CloudHub** (Optional - for multiple site connections)

**Features:**
- BGP dynamic routing (Optional)
- Static routing
- Accelerated VPN (Optional - using Global Accelerator)
- Certificate-based authentication (Optional - for Client VPN)

## Step-by-Step VPC Configuration

### Step 1: Create VPC

#### Using AWS Console:
1. Navigate to **VPC Console**
2. Click **Create VPC**
3. Configure:
   - **Name**: `my-vpc`
   - **IPv4 CIDR**: `10.0.0.0/16`
   - **IPv6 CIDR**: Optional (assign Amazon-provided IPv6 CIDR block)
   - **Tenancy**: Default (or Dedicated for compliance requirements - Optional)
   - **Tags**: Optional (for resource management)
4. Click **Create VPC**

#### Using AWS CLI:
```bash
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --amazon-provided-ipv6-cidr-block \
    --instance-tenancy default \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=my-vpc},{Key=Environment,Value=production}]'
```

#### Using Terraform:
```hcl
resource "aws_vpc" "main" {
  cidr_block                       = "10.0.0.0/16"
  assign_generated_ipv6_cidr_block = true  # Optional - for IPv6 support
  instance_tenancy                 = "default"
  enable_dns_hostnames             = true
  enable_dns_support               = true
  enable_network_address_usage_metrics = true  # Optional - for IP usage monitoring

  tags = {
    Name        = "my-vpc"
    Environment = "production"
  }
}
```

### Step 2: Create Internet Gateway

#### Using AWS Console:
1. Navigate to **Internet Gateways**
2. Click **Create Internet Gateway**
3. Set **Name**: `my-igw`
4. **IPv6 Support**: Optional (enable if VPC has IPv6 CIDR)
5. Click **Create**
6. Select the IGW and click **Actions** → **Attach to VPC**
7. Select your VPC and attach

#### Using AWS CLI:
```bash
# Create IGW
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=my-igw}]'

# Attach to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-1234567890abcdef0 \
    --vpc-id vpc-12345678
```

#### Using Terraform:
```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "my-igw"
    Environment = "production"
  }
}

# Optional: Egress-only Internet Gateway for IPv6
resource "aws_egress_only_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "my-eigw"
  }
}
```

### Step 3: Create Subnets

#### Public Subnet Configuration:
```bash
# Create public subnet
aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-west-2a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-1}]'

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute \
    --subnet-id subnet-12345678 \
    --map-public-ip-on-launch
```

#### Private Subnet Configuration:
```bash
# Create private subnet
aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.2.0/24 \
    --availability-zone us-west-2a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-subnet-1}]'
```

#### Terraform Example:
```hcl
# Public Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-west-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-1"
    Type = "Public"
  }
}

# Private Subnet
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-west-2a"

  tags = {
    Name = "private-subnet-1"
    Type = "Private"
  }
}
```

### Step 4: Create NAT Gateway

#### Using AWS Console:
1. Navigate to **NAT Gateways**
2. Click **Create NAT Gateway**
3. Configure:
   - **Subnet**: Select public subnet
   - **Connectivity**: Public
   - **Elastic IP**: Allocate new EIP

#### Using AWS CLI:
```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Create NAT Gateway
aws ec2 create-nat-gateway \
    --subnet-id subnet-12345678 \
    --allocation-id eipalloc-12345678 \
    --tag-specifications 'ResourceType=nat-gateway,Tags=[{Key=Name,Value=my-nat-gateway}]'
```

#### Using Terraform:
```hcl
# Elastic IP for NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
  
  tags = {
    Name = "nat-eip"
  }
}

# NAT Gateway
resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "my-nat-gateway"
  }

  depends_on = [aws_internet_gateway.main]
}
```

### Step 5: Configure Route Tables

#### Public Route Table:
```bash
# Create public route table
aws ec2 create-route-table \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-rt}]'

# Add route to Internet Gateway
aws ec2 create-route \
    --route-table-id rtb-12345678 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-1234567890abcdef0

# Associate with public subnet
aws ec2 associate-route-table \
    --route-table-id rtb-12345678 \
    --subnet-id subnet-12345678
```

#### Private Route Table:
```bash
# Create private route table
aws ec2 create-route-table \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=private-rt}]'

# Add route to NAT Gateway
aws ec2 create-route \
    --route-table-id rtb-87654321 \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id nat-1234567890abcdef0

# Associate with private subnet
aws ec2 associate-route-table \
    --route-table-id rtb-87654321 \
    --subnet-id subnet-87654321
```

#### Terraform Example:
```hcl
# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-rt"
  }
}

# Private Route Table
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = {
    Name = "private-rt"
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  subnet_id      = aws_subnet.private.id
  route_table_id = aws_route_table.private.id
}
```

### Step 6: Create Security Groups

#### Web Server Security Group:
```bash
# Create security group
aws ec2 create-security-group \
    --group-name web-sg \
    --description "Security group for web servers" \
    --vpc-id vpc-12345678

# Allow HTTP inbound
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Allow HTTPS inbound
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# Allow SSH inbound
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 22 \
    --cidr 10.0.0.0/16
```

#### Database Security Group:
```bash
# Create database security group
aws ec2 create-security-group \
    --group-name db-sg \
    --description "Security group for database servers" \
    --vpc-id vpc-12345678

# Allow MySQL/Aurora inbound from web security group
aws ec2 authorize-security-group-ingress \
    --group-id sg-87654321 \
    --protocol tcp \
    --port 3306 \
    --source-group sg-12345678
```

#### Terraform Example:
```hcl
# Web Server Security Group
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}

# Database Security Group
resource "aws_security_group" "database" {
  name        = "db-sg"
  description = "Security group for database servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "db-sg"
  }
}
```

### Step 7: Configure Network ACLs

#### Custom NACL Example:
```bash
# Create custom NACL
aws ec2 create-network-acl \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=custom-nacl}]'

# Allow HTTP inbound
aws ec2 create-network-acl-entry \
    --network-acl-id acl-12345678 \
    --rule-number 100 \
    --protocol tcp \
    --rule-action allow \
    --port-range From=80,To=80 \
    --cidr-block 0.0.0.0/0

# Allow HTTPS inbound
aws ec2 create-network-acl-entry \
    --network-acl-id acl-12345678 \
    --rule-number 110 \
    --protocol tcp \
    --rule-action allow \
    --port-range From=443,To=443 \
    --cidr-block 0.0.0.0/0

# Allow ephemeral ports outbound
aws ec2 create-network-acl-entry \
    --network-acl-id acl-12345678 \
    --rule-number 100 \
    --protocol tcp \
    --rule-action allow \
    --port-range From=1024,To=65535 \
    --cidr-block 0.0.0.0/0 \
    --egress
```

### Step 8: Create VPC Endpoints

#### S3 Gateway Endpoint:
```bash
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-12345678 \
    --service-name com.amazonaws.us-west-2.s3 \
    --route-table-ids rtb-12345678 rtb-87654321 \
    --policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:*",
                "Resource": "*"
            }
        ]
    }'
```

#### EC2 Interface Endpoint:
```bash
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-12345678 \
    --service-name com.amazonaws.us-west-2.ec2 \
    --vpc-endpoint-type Interface \
    --subnet-ids subnet-87654321 \
    --security-group-ids sg-12345678 \
    --private-dns-enabled
```

## Advanced VPC Features

### VPC Flow Logs Configuration

#### Enable Flow Logs for VPC:
```bash
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-12345678 \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name VPCFlowLogs \
    --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole \
    --log-format '${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${windowstart} ${windowend} ${action}'  # Optional - custom format
```

#### VPC Flow Logs to S3 (Optional):
```bash
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-12345678 \
    --traffic-type ALL \
    --log-destination-type s3 \
    --log-destination arn:aws:s3:::my-flow-logs-bucket/flow-logs/ \
    --log-format '${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${windowstart} ${windowend} ${action}' \
    --destination-options FileFormat=parquet,HiveCompatiblePartitions=true,PerHourPartition=true  # Optional - for cost optimization
```

#### Flow Log Record Format:
```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes windowstart windowend action flowlogstatus

# Optional custom fields can include:
# vpc-id subnet-id instance-id tcp-flags type pkt-srcaddr pkt-dstaddr region az-id sublocation-type sublocation-id pkt-src-aws-service pkt-dst-aws-service flow-direction traffic-path
```

### VPC Peering Configuration

#### Create VPC Peering Connection:
```bash
# Create peering connection
aws ec2 create-vpc-peering-connection \
    --vpc-id vpc-12345678 \
    --peer-vpc-id vpc-87654321 \
    --peer-region us-west-2 \
    --peer-owner-id 123456789012  # Optional - for cross-account peering

# Accept peering connection
aws ec2 accept-vpc-peering-connection \
    --vpc-peering-connection-id pcx-1234567890abcdef0

# Modify peering connection options (Optional)
aws ec2 modify-vpc-peering-connection-options \
    --vpc-peering-connection-id pcx-1234567890abcdef0 \
    --requester-peering-connection-options AllowDnsResolutionFromRemoteVpc=true \
    --accepter-peering-connection-options AllowDnsResolutionFromRemoteVpc=true

# Add routes to route tables
aws ec2 create-route \
    --route-table-id rtb-12345678 \
    --destination-cidr-block 10.1.0.0/16 \
    --vpc-peering-connection-id pcx-1234567890abcdef0
```

### Transit Gateway Setup (Optional)

#### Create Transit Gateway:
```bash
aws ec2 create-transit-gateway \
    --description "Main transit gateway" \
    --options AmazonSideAsn=64512,AutoAcceptSharedAttachments=enable,DefaultRouteTableAssociation=enable,DefaultRouteTablePropagation=enable,DnsSupport=enable,VpnEcmpSupport=enable,MulticastSupport=enable  # Optional multicast support

# Attach VPC to Transit Gateway
aws ec2 create-transit-gateway-vpc-attachment \
    --transit-gateway-id tgw-12345678 \
    --vpc-id vpc-12345678 \
    --subnet-ids subnet-12345678

# Optional: Create custom route table for Transit Gateway
aws ec2 create-transit-gateway-route-table \
    --transit-gateway-id tgw-12345678 \
    --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=custom-tgw-rt}]'

# Optional: Create Transit Gateway Connect attachment for SD-WAN appliances
aws ec2 create-transit-gateway-connect \
    --transport-attachment-id tgw-attach-12345678 \
    --options Protocol=gre
```

## Additional Optional VPC Components

### 17. AWS PrivateLink (Optional)
Service for private connectivity to services across different accounts and VPCs.

**Use Cases:**
- Expose services to other AWS accounts
- Connect to third-party SaaS applications
- Private connectivity without internet exposure
- Network Load Balancer integration (Optional)
- Gateway Load Balancer integration (Optional)

### 18. AWS VPC Lattice (Optional - New Service)
Application networking service for connecting and securing services across VPCs and accounts.

**Features:**
- Service mesh capabilities
- Multi-account service connectivity
- Built-in security and observability
- Application-level load balancing (Optional)

### 19. AWS Local Zones (Optional)
Infrastructure deployments that place compute, storage, and other services closer to end-users.

**Benefits:**
- Ultra-low latency applications
- Real-time gaming and media streaming
- Machine learning inference at the edge
- Dedicated subnet types (Optional)

### 20. AWS Wavelength (Optional)
Infrastructure for 5G networks that enables ultra-low latency applications.

**Use Cases:**
- AR/VR applications
- Real-time multiplayer gaming
- Industrial automation
- Connected vehicle applications
- Carrier IP addresses (Optional)

### 21. VPC Sharing (Optional)
Share VPC subnets with other AWS accounts in the same organization.

**Benefits:**
- Centralized network management
- Cost optimization
- Simplified security management
- Cross-account resource deployment (Optional)

## Best Practices

### 1. IP Address Planning
- **Use RFC 1918 Private Ranges**:
  - 10.0.0.0/8 (10.0.0.0 to 10.255.255.255)
  - 172.16.0.0/12 (172.16.0.0 to 172.31.255.255)
  - 192.168.0.0/16 (192.168.0.0 to 192.168.255.255)
- **Reserve IP Space**: Plan for future growth
- **Avoid Overlapping**: Ensure no CIDR conflicts with on-premises or other VPCs

### 2. Subnet Design
- **Multi-AZ Deployment**: Distribute subnets across multiple AZs
- **Subnet Types**: 
  - Public: Web servers, load balancers
  - Private: Application servers, databases
  - Isolated: Sensitive databases, internal services
- **Subnet Sizing**: Use /24 for most use cases (251 usable IPs)

### 3. Security Best Practices
- **Principle of Least Privilege**: Grant minimum required access
- **Defense in Depth**: Use both Security Groups and NACLs
- **Security Group Rules**:
  - Reference other security groups instead of IP ranges
  - Use specific ports instead of port ranges
  - Regularly audit and clean up unused rules
- **Network Monitoring**: Enable VPC Flow Logs

### 4. High Availability
- **Multi-AZ Design**: Deploy across at least 2 AZs
- **NAT Gateway**: One per AZ for redundancy
- **Load Balancers**: Use ALB/NLB for distribution
- **Auto Scaling**: Configure for automatic scaling

### 5. Cost Optimization
- **NAT Gateway**: Consider NAT instances for low-traffic scenarios
- **VPC Endpoints**: Use for AWS service communication
- **Data Transfer**: Monitor and optimize cross-AZ traffic
- **Elastic IPs**: Release unused EIPs

### 6. Monitoring and Troubleshooting
- **CloudWatch Metrics**: Monitor VPC components
- **VPC Flow Logs**: Analyze traffic patterns
- **AWS Config**: Track configuration changes
- **Network Insights**: Use Reachability Analyzer

### 7. Naming Conventions
```
# Resource naming pattern
<environment>-<application>-<component>-<az>

# Examples
prod-webapp-public-subnet-1a
prod-webapp-private-subnet-1b
prod-webapp-web-sg
prod-webapp-db-sg
```

### 8. Tagging Strategy
```json
{
  "Environment": "Production",
  "Application": "WebApp",
  "Owner": "DevOps Team",
  "CostCenter": "IT-001",
  "Project": "CustomerPortal",
  "Backup": "Daily"
}
```

## Complete Terraform Example

```hcl
# Complete VPC Infrastructure
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Variables
variable "aws_region" {
  default = "us-west-2"
}

variable "vpc_cidr" {
  default = "10.0.0.0/16"
}

variable "availability_zones" {
  default = ["us-west-2a", "us-west-2b"]
}

# Data Sources
data "aws_availability_zones" "available" {
  state = "available"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "main-vpc"
    Environment = "production"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# Public Subnets
resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "Public"
  }
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "private-subnet-${count.index + 1}"
    Type = "Private"
  }
}

# Elastic IPs for NAT Gateways
resource "aws_eip" "nat" {
  count  = length(var.availability_zones)
  domain = "vpc"

  tags = {
    Name = "nat-eip-${count.index + 1}"
  }
}

# NAT Gateways
resource "aws_nat_gateway" "main" {
  count         = length(var.availability_zones)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "nat-gateway-${count.index + 1}"
  }

  depends_on = [aws_internet_gateway.main]
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-rt"
  }
}

resource "aws_route_table" "private" {
  count  = length(var.availability_zones)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = {
    Name = "private-rt-${count.index + 1}"
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = length(var.availability_zones)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = length(var.availability_zones)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

# VPC Endpoints
resource "aws_vpc_endpoint" "s3" {
  vpc_id          = aws_vpc.main.id
  service_name    = "com.amazonaws.${var.aws_region}.s3"
  route_table_ids = concat([aws_route_table.public.id], aws_route_table.private[*].id)

  tags = {
    Name = "s3-endpoint"
  }
}

# Security Groups
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.vpc_cidr]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}

# Outputs
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}

output "security_group_web_id" {
  value = aws_security_group.web.id
}
```

This comprehensive guide covers all VPC components with detailed explanations, configurations, and best practices for building robust, secure, and scalable AWS network infrastructure.
