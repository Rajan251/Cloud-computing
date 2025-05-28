# Production-Ready ALB & ASG Setup - Real-Time Project Guide

## Why Application Load Balancer (ALB) is Most Used in Production

### Real-Time Usage Statistics
- **80%+ of production projects** use ALB over NLB or CLB
- **Layer 7 routing** needed for modern web applications
- **Microservices architecture** requires path-based routing
- **Container workloads** (ECS, EKS) prefer ALB integration

### ALB vs Others in Production:
- **ALB**: Web apps, APIs, microservices (Most Common)
- **NLB**: Gaming, IoT, ultra-low latency (10-15% usage)
- **CLB**: Legacy applications (5% and decreasing)

---

## Production-Ready ALB Creation

### Pre-Production Checklist
- [ ] Multi-AZ VPC setup completed
- [ ] Security groups planned for least privilege
- [ ] SSL certificates ready (ACM or imported)
- [ ] Monitoring and logging strategy defined
- [ ] Disaster recovery plan documented

### Step 1: VPC and Network Planning (Production Critical)
```
Production Network Design:
├── VPC: 10.0.0.0/16
├── Public Subnets (ALB):
│   ├── us-east-1a: 10.0.1.0/24
│   ├── us-east-1b: 10.0.2.0/24
│   └── us-east-1c: 10.0.3.0/24
└── Private Subnets (EC2):
    ├── us-east-1a: 10.0.10.0/24
    ├── us-east-1b: 10.0.20.0/24
    └── us-east-1c: 10.0.30.0/24
```

**Production Explanation**: 
- **3 AZs minimum** for enterprise-grade availability
- **Separate subnets** for ALB (public) and instances (private)
- **CIDR planning** allows for future expansion

**Best Practice**: Reserve IP ranges for future services (databases, cache, etc.)

### Step 2: Security Groups (Production Security)

#### ALB Security Group (Production)
```
Name: prod-alb-sg
Inbound Rules:
├── HTTPS (443) from 0.0.0.0/0 (Public access)
├── HTTP (80) from 0.0.0.0/0 (Redirect to HTTPS)
└── Custom (8080) from Corporate IP range (Admin access)

Outbound Rules:
└── All traffic to Application Security Group
```

#### Application Security Group (Production)
```
Name: prod-app-sg
Inbound Rules:
├── HTTP (80) from ALB Security Group only
├── HTTPS (443) from ALB Security Group only
├── SSH (22) from Bastion Host SG only
└── Custom App Port from ALB SG only

Outbound Rules:
├── HTTPS (443) to 0.0.0.0/0 (Package updates, API calls)
├── MySQL (3306) to Database SG
└── Redis (6379) to Cache SG
```

**Production Explanation**:
- **Least privilege principle**: Only necessary ports open
- **Source-based rules**: Traffic only from trusted sources
- **No direct SSH**: Use bastion host or Systems Manager

### Step 3: SSL/TLS Certificate Setup (Production Security)

#### Using AWS Certificate Manager (Recommended)
1. **Request Certificate**:
   - Domain: `*.yourcompany.com`
   - Validation: DNS validation (automated renewal)
   - Include multiple domains: `yourcompany.com`, `api.yourcompany.com`

2. **DNS Validation**:
   - Add CNAME records to Route 53 or external DNS
   - Wait for validation (5-30 minutes)

**Production Explanation**:
- **Wildcard certificates** cover all subdomains
- **DNS validation** enables auto-renewal
- **Multiple domains** support different environments

**Best Practice**: Use separate certificates for different environments (dev, staging, prod)

### Step 4: Create Production ALB

#### Basic Configuration (Production)
```
Name: prod-web-alb
Scheme: Internet-facing
IP Address Type: IPv4
Deletion Protection: Enabled ✓
```

**Production Explanation**:
- **Deletion protection** prevents accidental deletion
- **Descriptive naming** follows company naming convention
- **Internet-facing** for public web applications

#### Network Mapping (Production)
```
VPC: prod-vpc
Availability Zones: All 3 AZs
Subnets: Public subnets only
├── prod-public-1a (10.0.1.0/24)
├── prod-public-1b (10.0.2.0/24)
└── prod-public-1c (10.0.3.0/24)
```

**Production Explanation**:
- **All AZs** for maximum availability
- **Public subnets** for internet-facing ALB
- **Consistent subnet naming** for operations team

#### Security Group Assignment (Production)
```
Security Group: prod-alb-sg
Custom Rules:
├── HTTPS 443 from 0.0.0.0/0
├── HTTP 80 from 0.0.0.0/0 (for redirect)
└── Health Check port from ALB to targets
```

### Step 5: Listeners Configuration (Production)

#### HTTP Listener (Redirect to HTTPS)
```
Protocol: HTTP
Port: 80
Default Action: Redirect to HTTPS
├── Protocol: HTTPS
├── Port: 443
└── Status Code: 301 (Permanent Redirect)
```

#### HTTPS Listener (Main Traffic)
```
Protocol: HTTPS
Port: 443
SSL Certificate: ACM Certificate (*.yourcompany.com)
Security Policy: ELBSecurityPolicy-TLS-1-2-2017-01
Default Action: Forward to prod-web-tg
```

**Production Explanation**:
- **Force HTTPS** for security compliance
- **Latest TLS policy** for security standards
- **Permanent redirect** for SEO benefits

**Best Practice**: Use TLS 1.2 minimum, consider TLS 1.3 for new applications

### Step 6: Target Group Configuration (Production)

#### Primary Target Group
```
Name: prod-web-tg
Target Type: Instance
Protocol: HTTP
Port: 80
VPC: prod-vpc

Health Check Settings:
├── Protocol: HTTP
├── Path: /health
├── Healthy Threshold: 2
├── Unhealthy Threshold: 3
├── Timeout: 5 seconds
├── Interval: 10 seconds
└── Success Codes: 200,201,202
```

**Production Explanation**:
- **Dedicated health endpoint** (`/health`) separate from main app
- **Fast health checks** (10s interval) for quick failover
- **Multiple success codes** for flexible responses
- **Conservative thresholds** prevent flapping

#### Advanced Health Check (Production)
```
Health Check Endpoint Response:
{
  "status": "healthy",
  "timestamp": "2024-01-15T10:30:00Z",
  "database": "connected",
  "cache": "connected",
  "version": "1.2.3"
}
```

**Best Practice**: Health check should verify all critical dependencies

### Step 7: Advanced Routing Rules (Production)

#### Path-Based Routing (Microservices)
```
Priority 100: /api/* → Forward to prod-api-tg
Priority 200: /admin/* → Forward to prod-admin-tg
Priority 300: /static/* → Forward to prod-static-tg
Default: /* → Forward to prod-web-tg
```

#### Host-Based Routing (Multi-tenant)
```
Priority 50: api.yourcompany.com → Forward to prod-api-tg
Priority 60: admin.yourcompany.com → Forward to prod-admin-tg
Default: yourcompany.com → Forward to prod-web-tg
```

**Production Explanation**:
- **Lower numbers = higher priority**
- **Specific rules first**, general rules last
- **Separate target groups** for different services

### Step 8: Monitoring and Logging (Production Critical)

#### Enable Access Logs
```
S3 Bucket: prod-alb-access-logs-{account-id}
Prefix: alb-logs/
Lifecycle Policy: 
├── 30 days: Standard storage
├── 90 days: IA storage
└── 365 days: Delete
```

#### CloudWatch Metrics to Monitor
```
Critical Metrics:
├── TargetResponseTime (< 200ms target)
├── HTTPCode_Target_4XX_Count (< 1% error rate)
├── HTTPCode_Target_5XX_Count (< 0.1% error rate)
├── HealthyHostCount (>= 2 hosts minimum)
└── RequestCount (for capacity planning)
```

#### CloudWatch Alarms (Production)
```
High Error Rate:
├── Metric: HTTPCode_Target_5XX_Count
├── Threshold: > 10 errors in 5 minutes
├── Action: SNS → PagerDuty → On-call engineer

High Response Time:
├── Metric: TargetResponseTime
├── Threshold: > 500ms for 3 minutes
├── Action: SNS → DevOps team

No Healthy Targets:
├── Metric: HealthyHostCount
├── Threshold: < 1 for 1 minute
├── Action: SNS → Immediate alert
```

---

## Production-Ready Auto Scaling Group

### Step 1: Launch Template (Production)

#### Comprehensive Launch Template
```
Name: prod-web-lt-v1
AMI: ami-0abcdef123456789 (Custom hardened AMI)
Instance Type: t3.medium (or m5.large for high traffic)
Key Pair: prod-web-keypair
Security Groups: prod-app-sg

Advanced Settings:
├── IAM Role: prod-ec2-role
├── User Data: Bootstrap script
├── EBS Optimization: Enabled
├── Detailed Monitoring: Enabled
├── Instance Metadata: IMDSv2 required
└── Termination Protection: Disabled (for ASG)
```

#### Production User Data Script
```bash
#!/bin/bash
# Production bootstrap script
yum update -y
yum install -y amazon-cloudwatch-agent

# Install application
aws s3 cp s3://prod-deployment-bucket/app-v1.2.3.tar.gz /tmp/
tar -xzf /tmp/app-v1.2.3.tar.gz -C /opt/

# Configure CloudWatch agent
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -s -c s3://prod-config-bucket/cloudwatch-config.json

# Configure log rotation
echo '/var/log/app/*.log {
  daily
  rotate 7
  compress
  missingok
  notifempty
}' > /etc/logrotate.d/app

# Start application
systemctl enable app-service
systemctl start app-service

# Signal ASG that instance is ready
/opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource AutoScalingGroup --region ${AWS::Region}
```

**Production Explanation**:
- **Versioned deployments** using S3 artifacts
- **CloudWatch agent** for detailed monitoring
- **Log management** with rotation
- **Signal when ready** for proper ASG integration

### Step 2: Auto Scaling Group Configuration (Production)

#### Basic ASG Settings
```
Name: prod-web-asg
Launch Template: prod-web-lt-v1
VPC: prod-vpc
Subnets: All private subnets (3 AZs)
├── prod-private-1a
├── prod-private-1b
└── prod-private-1c
```

#### Capacity Settings (Production)
```
Desired Capacity: 6 instances
Minimum Capacity: 3 instances (1 per AZ minimum)
Maximum Capacity: 18 instances
```

**Production Explanation**:
- **Even numbers** for balanced distribution
- **Minimum 1 per AZ** for availability
- **3x scaling headroom** for traffic spikes

#### Load Balancer Integration
```
Target Groups: prod-web-tg
Health Check Type: ELB (not just EC2)
Health Check Grace Period: 300 seconds
```

**Production Explanation**:
- **ELB health check** considers application health
- **Grace period** allows app startup time

### Step 3: Production Scaling Policies

#### Target Tracking Scaling (Primary)
```
Policy Name: prod-cpu-scaling
Metric: Average CPU Utilization
Target Value: 70%
Scale-out Cooldown: 300 seconds
Scale-in Cooldown: 600 seconds
```

#### Request-Based Scaling (Secondary)
```
Policy Name: prod-request-scaling
Metric: ALB Request Count Per Target
Target Value: 1000 requests per minute per instance
```

#### Custom Metric Scaling (Advanced)
```
Policy Name: prod-response-time-scaling
Metric: ALB Target Response Time
Target Value: 200ms
CloudWatch Alarm: Custom metric from application
```

**Production Explanation**:
- **Multiple metrics** prevent over/under scaling
- **Conservative targets** prevent thrashing
- **Longer scale-in** cooldown prevents premature termination

### Step 4: Production Monitoring Setup

#### Instance-Level Monitoring
```
CloudWatch Agent Metrics:
├── Memory utilization
├── Disk utilization
├── Network performance
├── Application-specific metrics
└── Custom business metrics
```

#### ASG-Level Monitoring
```
Key Metrics:
├── GroupDesiredCapacity
├── GroupInServiceInstances
├── GroupTotalInstances
├── Scaling activities
└── Launch/termination events
```

#### Business-Level Monitoring
```
Custom Metrics:
├── Active user sessions
├── Transaction rate
├── Revenue per minute
├── Error rates by endpoint
└── Database connection pool
```

---

## Production Best Practices & Real-Time Recommendations

### 1. Security Best Practices
```
✓ Use WAF with ALB for application protection
✓ Enable GuardDuty for threat detection
✓ Regular security group audits
✓ Rotate SSL certificates automatically
✓ Use Systems Manager for patch management
✓ Enable VPC Flow Logs
✓ Implement least privilege IAM roles
```

### 2. High Availability Best Practices
```
✓ Deploy across minimum 3 AZs
✓ Use multiple target groups for blue/green deployments
✓ Implement circuit breakers in application
✓ Set up cross-region disaster recovery
✓ Use Route 53 health checks for DNS failover
✓ Regular disaster recovery testing
```

### 3. Performance Best Practices
```
✓ Use connection draining (300s recommended)
✓ Enable sticky sessions only if required
✓ Implement proper caching strategy
✓ Use CloudFront for static content
✓ Optimize health check frequency
✓ Right-size instances based on metrics
```

### 4. Cost Optimization Best Practices
```
✓ Use Spot instances for non-critical workloads (ASG mixed instances)
✓ Implement scheduled scaling for predictable patterns
✓ Use Reserved Instances for baseline capacity
✓ Regular right-sizing analysis
✓ Clean up unused load balancers and target groups
✓ Use S3 lifecycle policies for access logs
```

### 5. Operational Best Practices
```
✓ Implement Infrastructure as Code (CloudFormation/Terraform)
✓ Use Blue/Green deployments
✓ Automate AMI creation and updates
✓ Implement proper tagging strategy
✓ Set up centralized logging (ELK stack)
✓ Use Parameter Store for configuration management
✓ Implement automated backup strategies
```

### 6. Real-Time Project Checklist

#### Before Go-Live
- [ ] Load testing completed (2x expected traffic)
- [ ] Security penetration testing done
- [ ] Disaster recovery procedures tested
- [ ] Monitoring and alerting configured
- [ ] On-call procedures documented
- [ ] Scaling policies validated
- [ ] SSL certificates validated and monitored
- [ ] Access logs and audit trails enabled

#### Post Go-Live Monitoring
- [ ] Daily health check of all components
- [ ] Weekly performance review
- [ ] Monthly cost optimization review
- [ ] Quarterly security audit
- [ ] Bi-annual disaster recovery testing

---

## Common Production Issues and Solutions

### Issue 1: Intermittent 502/503 Errors
**Root Cause**: Health check path returning errors
**Solution**: 
- Check application health endpoint
- Verify security group rules
- Increase health check timeout
- Review application logs

### Issue 2: Slow Scale-Out During Traffic Spikes
**Root Cause**: Conservative scaling policies
**Solution**:
- Reduce scale-out cooldown
- Lower CPU threshold
- Add predictive scaling
- Use warm pool for faster instance availability

### Issue 3: High ALB Costs
**Root Cause**: Too many ALBs or inefficient routing
**Solution**:
- Consolidate ALBs using host/path routing
- Use shared ALBs across environments
- Implement proper request routing
- Monitor LCU (Load Balancer Capacity Units) usage

### Issue 4: SSL Certificate Renewal Failures
**Root Cause**: DNS validation issues
**Solution**:
- Use Route 53 for automatic DNS validation
- Set up CloudWatch alarms for certificate expiry
- Implement automated certificate renewal testing
- Maintain backup certificates
