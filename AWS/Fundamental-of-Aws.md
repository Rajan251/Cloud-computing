# AWS Services Quick Reference Guide

## Compute Services
- **EC2 (Elastic Compute Cloud)**
  - Virtual servers in the cloud
  - Scalable computing capacity
  - Multiple instance types (general purpose, compute optimized, memory optimized)
  - Pay only for what you use

- **Lambda**
  - Run code without managing servers
  - Pay only for compute time used
  - Automatic scaling
  - Supports multiple programming languages (Python, Node.js, Java, etc.)

- **ECS (Elastic Container Service)**
  - Run and manage Docker containers
  - Easily scale containerized applications
  - Integrates with other AWS services

- **EKS (Elastic Kubernetes Service)**
  - Managed Kubernetes service
  - Run Kubernetes without installing or maintaining your own Kubernetes control plane
  - Compatible with existing Kubernetes tools

- **Fargate**
  - Serverless compute engine for containers
  - Works with ECS and EKS
  - No need to manage servers or clusters
  - Pay only for resources required to run containers

## Storage Services
- **S3 (Simple Storage Service)**
  - Object storage service
  - Store and retrieve any amount of data
  - Highly scalable and durable
  - Different storage classes for different needs (Standard, Infrequent Access, Glacier)

- **EBS (Elastic Block Store)**
  - Block-level storage volumes for EC2 instances
  - Independent of instance lifecycles
  - Different volume types (SSD, HDD)
  - Snapshots for backup

- **EFS (Elastic File System)**
  - Fully managed file storage for EC2
  - Automatically grows and shrinks as you add/remove files
  - Shared access across multiple instances

- **FSx**
  - Fully managed file systems
  - Windows File Server for Windows applications
  - Lustre for high-performance computing
  - NetApp ONTAP for enterprise applications

- **Snow Family**
  - Physical devices to transfer data in/out of AWS
  - Snowcone: small, portable computing device
  - Snowball: suitcase-sized data transfer device
  - Snowmobile: shipping container of storage (exabyte-scale)

## Database Services
- **RDS (Relational Database Service)**
  - Managed relational database service
  - Supports multiple engines (MySQL, PostgreSQL, Oracle, SQL Server, MariaDB)
  - Automated backups and patching

- **Aurora**
  - MySQL and PostgreSQL compatible database
  - 5x faster than standard MySQL, 3x faster than PostgreSQL
  - Automatic scaling and high availability

- **DynamoDB**
  - Fully managed NoSQL database
  - Single-digit millisecond performance
  - Serverless with automatic scaling
  - Supports both document and key-value data models

- **ElastiCache**
  - In-memory caching service
  - Improves application performance
  - Supports Redis and Memcached
  - Easy to deploy and scale

- **DocumentDB**
  - MongoDB-compatible document database
  - Fully managed with automatic scaling
  - Highly available with replicas across multiple zones

- **Neptune**
  - Fully managed graph database
  - Support for property graph and RDF
  - High availability with up to 15 read replicas
  - Good for relationship-heavy data (social networks, recommendations)

## Networking & Content Delivery
- **VPC (Virtual Private Cloud)**
  - Isolated cloud resources
  - Define your own IP address range
  - Create subnets, route tables, network gateways
  - Control security with Security Groups and Network ACLs

- **Route 53**
  - Highly available DNS service
  - Domain registration
  - Health checks and monitoring
  - Various routing policies (simple, weighted, latency-based)

- **CloudFront**
  - Global content delivery network (CDN)
  - Speeds up distribution of static and dynamic content
  - Low latency and high transfer speeds
  - Integrates with AWS Shield for DDoS protection

- **API Gateway**
  - Create, publish, and manage APIs
  - RESTful and WebSocket APIs
  - Traffic management and monitoring
  - API versioning and authentication

- **Direct Connect**
  - Dedicated network connection from your premises to AWS
  - Reduces network costs and increases bandwidth
  - More consistent network experience
  - Private connectivity to your VPC

## Security, Identity & Compliance
- **IAM (Identity and Access Management)**
  - Manage access to AWS services and resources
  - Create and manage users and groups
  - Fine-grained permissions
  - Multi-factor authentication

- **Cognito**
  - User identity and data synchronization
  - User pools for authentication
  - Identity pools for temporary AWS access
  - Social identity provider integration

- **WAF (Web Application Firewall)**
  - Protects web applications from common attacks
  - Create security rules to block common attack patterns
  - Monitor web requests
  - Integrates with CloudFront and Application Load Balancer

- **Shield**
  - DDoS protection service
  - Standard: automatically included for all AWS customers
  - Advanced: 24/7 DDoS response team, detailed attack diagnostics

- **Certificate Manager**
  - Provision and manage SSL/TLS certificates
  - Free public certificates
  - Automatic renewal
  - Integration with AWS services (CloudFront, API Gateway)

## Management & Governance
- **CloudWatch**
  - Monitoring and observability service
  - Collect and track metrics
  - Collect and monitor log files
  - Set alarms and create dashboards

- **CloudTrail**
  - Tracks user activity and API usage
  - Records AWS account activity
  - Enables security analysis and compliance auditing
  - History of events for your account

- **Config**
  - Assess, audit, and evaluate configurations
  - Track configuration changes
  - Evaluate resources against best practices
  - Simplifies compliance auditing

- **CloudFormation**
  - Infrastructure as code
  - Create and manage resources with templates
  - Automate the setup of entire environments
  - Version control for infrastructure

- **Systems Manager**
  - Operational management for AWS resources
  - Group resources by application
  - View operational data
  - Automate operational tasks

## Application Integration
- **SQS (Simple Queue Service)**
  - Fully managed message queuing
  - Decouples application components
  - Guarantees message delivery
  - Scales automatically

- **SNS (Simple Notification Service)**
  - Fully managed pub/sub messaging
  - Send messages to multiple subscribers
  - Supports multiple protocols (HTTP, email, SMS, SQS)
  - Topic-based message filtering

- **EventBridge**
  - Serverless event bus
  - Connect applications with real-time data
  - Route events between AWS services
  - Build event-driven architectures

- **Step Functions**
  - Visual workflow service
  - Coordinate multiple AWS services into applications
  - Manages state, checkpoints, and restarts
  - Tracks each step for troubleshooting
