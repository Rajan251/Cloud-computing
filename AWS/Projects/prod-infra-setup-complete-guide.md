# Production-Ready AWS Infrastructure Setup
## Complete 3-Tier Architecture with EC2, Database Servers, and Client VPN

## Architecture Overview

```
Internet Users → ALB → Application Servers (Private Subnet)
                           ↓
Client VPN Users ← → Database Servers (Private Subnet)
                           ↓
              NAT Gateway → Internet (for updates)
```

**Key Components:**
- **Application Servers**: EC2 instances running web application (private subnet)
- **Database Servers**: EC2 instances running MySQL/PostgreSQL (private subnet) 
- **Client VPN**: Direct secure access to both app and DB servers
- **Load Balancer**: ALB for application traffic from internet users

---

## Phase 1: Network Infrastructure Setup

### Step 1: VPC and Subnet Planning

#### Create Production VPC
```
VPC Configuration:
├── Name: prod-vpc
├── CIDR: 10.0.0.0/16
├── DNS Hostnames: Enabled
├── DNS Resolution: Enabled
└── Tenancy: Default
```

#### Create Subnets (Multi-AZ for High Availability)
```
Public Subnets (for ALB and NAT Gateway):
├── prod-public-1a: 10.0.1.0/24 (us-east-1a)
├── prod-public-1b: 10.0.2.0/24 (us-east-1b)
└── prod-public-1c: 10.0.3.0/24 (us-east-1c)

Private Subnets - Application Tier:
├── prod-app-1a: 10.0.10.0/24 (us-east-1a)
├── prod-app-1b: 10.0.20.0/24 (us-east-1b)
└── prod-app-1c: 10.0.30.0/24 (us-east-1c)

Private Subnets - Database Tier:
├── prod-db-1a: 10.0.40.0/24 (us-east-1a)
├── prod-db-1b: 10.0.50.0/24 (us-east-1b)
└── prod-db-1c: 10.0.60.0/24 (us-east-1c)
```

**Why this design:**
- **3-Tier Architecture**: Public, App, Database isolation
- **Multi-AZ**: High availability and disaster recovery
- **CIDR Planning**: Room for future expansion

### Step 2: Internet Gateway and NAT Gateway

#### Internet Gateway
```
Name: prod-igw
Attach to: prod-vpc
```

#### NAT Gateways (for outbound internet access)
```
NAT Gateway 1:
├── Name: prod-nat-1a
├── Subnet: prod-public-1a
├── Elastic IP: Allocate new
└── Purpose: App/DB servers internet access

NAT Gateway 2 (Optional for HA):
├── Name: prod-nat-1b  
├── Subnet: prod-public-1b
├── Elastic IP: Allocate new
└── Purpose: Backup internet access
```

**Cost Note**: Single NAT Gateway = $32/month. Add second for HA = $64/month total.

### Step 3: Route Tables

#### Public Route Table
```
Name: prod-public-rt
Routes:
├── 10.0.0.0/16 → Local
└── 0.0.0.0/0 → Internet Gateway

Associated Subnets:
├── prod-public-1a
├── prod-public-1b
└── prod-public-1c
```

#### Private Route Table (App Tier)
```
Name: prod-app-rt
Routes:
├── 10.0.0.0/16 → Local
└── 0.0.0.0/0 → NAT Gateway (prod-nat-1a)

Associated Subnets:
├── prod-app-1a
├── prod-app-1b
└── prod-app-1c
```

#### Private Route Table (DB Tier)
```
Name: prod-db-rt
Routes:
├── 10.0.0.0/16 → Local
└── 0.0.0.0/0 → NAT Gateway (prod-nat-1a)

Associated Subnets:
├── prod-db-1a
├── prod-db-1b
└── prod-db-1c
```

---

## Phase 2: Security Groups Configuration

### Application Server Security Group
```
Name: prod-app-sg
Description: Security group for application servers

Inbound Rules:
├── HTTP (80) from prod-alb-sg
├── HTTPS (443) from prod-alb-sg
├── SSH (22) from prod-client-vpn-sg
├── Custom App Port (8080) from prod-alb-sg
└── MySQL (3306) from prod-db-sg (if app connects to local DB)

Outbound Rules:
├── All Traffic (0.0.0.0/0) - for package updates
├── MySQL (3306) to prod-db-sg
└── HTTPS (443) to 0.0.0.0/0 - for API calls
```

### Database Server Security Group
```
Name: prod-db-sg
Description: Security group for database servers

Inbound Rules:
├── MySQL (3306) from prod-app-sg
├── MySQL (3306) from prod-client-vpn-sg
├── SSH (22) from prod-client-vpn-sg
├── PostgreSQL (5432) from prod-app-sg (if using PostgreSQL)
└── PostgreSQL (5432) from prod-client-vpn-sg

Outbound Rules:
├── HTTP (80) to 0.0.0.0/0 - for package updates
├── HTTPS (443) to 0.0.0.0/0 - for package updates
└── MySQL (3306) to prod-db-sg - for DB replication
```

### ALB Security Group
```
Name: prod-alb-sg
Description: Security group for Application Load Balancer

Inbound Rules:
├── HTTP (80) from 0.0.0.0/0
├── HTTPS (443) from 0.0.0.0/0
└── Custom (8080) from Corporate IP ranges (for health checks)

Outbound Rules:
└── All Traffic to prod-app-sg
```

### Client VPN Security Group
```
Name: prod-client-vpn-sg
Description: Security group for Client VPN users

Inbound Rules:
└── (No inbound rules needed)

Outbound Rules:
├── SSH (22) to prod-app-sg
├── SSH (22) to prod-db-sg
├── MySQL (3306) to prod-db-sg
├── PostgreSQL (5432) to prod-db-sg
├── HTTP (80) to prod-app-sg
└── HTTPS (443) to prod-app-sg
```

---

## Phase 3: Database Server Setup (EC2-based)

### Step 1: Launch Database Servers

#### Primary Database Server
```
Instance Configuration:
├── AMI: Amazon Linux 2 (latest)
├── Instance Type: t3.medium (2 vCPU, 4GB RAM)
├── Key Pair: prod-db-keypair
├── VPC: prod-vpc
├── Subnet: prod-db-1a (primary AZ)
├── Security Group: prod-db-sg
├── IAM Role: prod-db-role
├── User Data: Database bootstrap script
└── Storage: 
    ├── Root: 20GB GP3
    └── Data: 100GB GP3 (mounted as /data)
```

#### Secondary Database Server (for HA)
```
Instance Configuration:
├── AMI: Amazon Linux 2 (latest)
├── Instance Type: t3.medium
├── Subnet: prod-db-1b (different AZ)
├── Security Group: prod-db-sg
├── Storage: Same as primary
└── Purpose: Read replica or standby
```

### Step 2: Database Server Bootstrap Script

#### MySQL Setup Script
```bash
#!/bin/bash
# Database Server Bootstrap Script

# Update system
yum update -y
yum install -y mysql-server mysql aws-cli

# Create data directory
mkdir -p /data/mysql
chown mysql:mysql /data/mysql

# Configure MySQL
cat > /etc/my.cnf << 'EOF'
[mysqld]
datadir=/data/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Performance settings
innodb_buffer_pool_size=2G
innodb_log_file_size=256M
max_connections=200

# Security settings
bind-address=0.0.0.0
skip-networking=0

# Backup settings
log-bin=mysql-bin
server-id=1
binlog-format=ROW
EOF

# Initialize MySQL
mysql_install_db --user=mysql --datadir=/data/mysql
systemctl enable mysqld
systemctl start mysqld

# Get temporary password
TEMP_PASSWORD=$(grep 'temporary password' /var/log/mysqld.log | awk '{print $NF}')

# Secure MySQL installation
mysql -u root -p"$TEMP_PASSWORD" --connect-expired-password << 'EOF'
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourStrongPassword123!';
DELETE FROM mysql.user WHERE User='';
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';
FLUSH PRIVILEGES;
EOF

# Create application database and user
mysql -u root -p'YourStrongPassword123!' << 'EOF'
CREATE DATABASE prodapp;
CREATE USER 'appuser'@'%' IDENTIFIED BY 'AppUserPassword123!';
GRANT ALL PRIVILEGES ON prodapp.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
EOF

# Configure backup script
cat > /home/ec2-user/backup_db.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
mysqldump -u root -p'YourStrongPassword123!' --all-databases > /tmp/backup_$DATE.sql
aws s3 cp /tmp/backup_$DATE.sql s3://prod-db-backups/mysql/
rm /tmp/backup_$DATE.sql
EOF

chmod +x /home/ec2-user/backup_db.sh

# Schedule daily backups
echo "0 2 * * * /home/ec2-user/backup_db.sh" | crontab -

# Install CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm

# Configure CloudWatch agent for MySQL monitoring
cat > /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json << 'EOF'
{
  "metrics": {
    "namespace": "CWAgent",
    "metrics_collected": {
      "cpu": {
        "measurement": ["cpu_usage_idle", "cpu_usage_iowait"],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["used_percent"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/mysqld.log",
            "log_group_name": "/aws/ec2/mysql",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
EOF

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
```

### Step 3: Database Replication Setup (Master-Slave)

#### On Primary Server (Master)
```sql
-- Enable binary logging (already done in my.cnf)
CREATE USER 'replication'@'%' IDENTIFIED BY 'ReplicationPassword123!';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
FLUSH PRIVILEGES;

-- Get master status
SHOW MASTER STATUS;
-- Note: File and Position values
```

#### On Secondary Server (Slave)
```bash
# Update my.cnf with different server-id
sed -i 's/server-id=1/server-id=2/' /etc/my.cnf
systemctl restart mysqld
```

```sql
-- Configure slave
CHANGE MASTER TO
  MASTER_HOST='10.0.40.10',  -- Primary DB private IP
  MASTER_USER='replication',
  MASTER_PASSWORD='ReplicationPassword123!',
  MASTER_LOG_FILE='mysql-bin.000001',  -- From SHOW MASTER STATUS
  MASTER_LOG_POS=154;  -- From SHOW MASTER STATUS

START SLAVE;
SHOW SLAVE STATUS\G
```

---

## Phase 4: Application Server Setup

### Step 1: Launch Application Servers

#### Launch Template for App Servers
```
Name: prod-app-lt-v1
AMI: Amazon Linux 2
Instance Type: t3.medium (or m5.large for high traffic)
Key Pair: prod-app-keypair
Security Groups: prod-app-sg
IAM Role: prod-app-role

User Data: Application bootstrap script
Storage:
├── Root: 20GB GP3
└── App Data: 50GB GP3 (mounted as /app)
```

#### Auto Scaling Group
```
Name: prod-app-asg
Launch Template: prod-app-lt-v1
VPC: prod-vpc
Subnets: prod-app-1a, prod-app-1b, prod-app-1c
Desired: 2
Min: 1
Max: 6
Health Check: ELB
```

### Step 2: Application Server Bootstrap Script

#### Node.js Application Setup
```bash
#!/bin/bash
# Application Server Bootstrap Script

# Update system
yum update -y
yum install -y git docker nodejs npm nginx

# Create application directory
mkdir -p /app
chown ec2-user:ec2-user /app

# Install Node.js application (example)
cd /app
git clone https://github.com/your-company/your-app.git .
npm install

# Create application configuration
cat > /app/config/production.json << 'EOF'
{
  "port": 3000,
  "database": {
    "host": "10.0.40.10",
    "port": 3306,
    "database": "prodapp",
    "username": "appuser",
    "password": "AppUserPassword123!"
  },
  "redis": {
    "host": "10.0.40.20",
    "port": 6379
  }
}
EOF

# Create systemd service
cat > /etc/systemd/system/nodeapp.service << 'EOF'
[Unit]
Description=Node.js Application
After=network.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/app
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=10
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
EOF

# Configure Nginx as reverse proxy
cat > /etc/nginx/conf.d/app.conf << 'EOF'
server {
    listen 80;
    server_name _;
    
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
EOF

# Start services
systemctl enable nginx nodeapp
systemctl start nginx nodeapp

# Install CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm

# Configure monitoring
cat > /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json << 'EOF'
{
  "metrics": {
    "namespace": "CWAgent",
    "metrics_collected": {
      "cpu": {
        "measurement": ["cpu_usage_idle", "cpu_usage_iowait"],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["used_percent"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/nginx/access.log",
            "log_group_name": "/aws/ec2/nginx/access",
            "log_stream_name": "{instance_id}"
          },
          {
            "file_path": "/app/logs/app.log",
            "log_group_name": "/aws/ec2/app",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
EOF

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
```

---

## Phase 5: Client VPN Setup

### Step 1: Certificate Management

#### Generate Certificates (Production Method)
```bash
# Create certificate authority
openssl genrsa -out ca-key.pem 2048
openssl req -new -x509 -days 3650 -key ca-key.pem -out ca-cert.pem \
  -subj "/C=US/ST=CA/L=SanFrancisco/O=YourCompany/CN=VPN-CA"

# Create server certificate
openssl genrsa -out server-key.pem 2048
openssl req -new -key server-key.pem -out server.csr \
  -subj "/C=US/ST=CA/L=SanFrancisco/O=YourCompany/CN=server"
openssl x509 -req -days 3650 -in server.csr -CA ca-cert.pem \
  -CAkey ca-key.pem -CAcreateserial -out server-cert.pem

# Create client certificate template
openssl genrsa -out client-key.pem 2048
openssl req -new -key client-key.pem -out client.csr \
  -subj "/C=US/ST=CA/L=SanFrancisco/O=YourCompany/CN=client"
openssl x509 -req -days 3650 -in client.csr -CA ca-cert.pem \
  -CAkey ca-key.pem -CAcreateserial -out client-cert.pem

# Upload server certificate to ACM
aws acm import-certificate \
  --certificate fileb://server-cert.pem \
  --private-key fileb://server-key.pem \
  --certificate-chain fileb://ca-cert.pem \
  --region us-east-1
```

### Step 2: Create Client VPN Endpoint

#### VPN Endpoint Configuration
```
Name: prod-client-vpn
Client IPv4 CIDR: 10.1.0.0/16
Server Certificate ARN: arn:aws:acm:us-east-1:123456789012:certificate/abc123
Authentication Type: Mutual authentication
Client Certificate ARN: Same as server certificate
VPC: prod-vpc
Security Groups: prod-client-vpn-sg
Self-service portal: Enabled
Session timeout: 12 hours
```

#### DNS Configuration
```
DNS Servers:
├── Primary: 10.0.0.2 (VPC DNS)
└── Secondary: 8.8.8.8 (Google DNS)
Domain Name: ec2.internal
```

### Step 3: Configure Authorization Rules

#### Allow Access to VPC Resources
```
Target Network: 10.0.0.0/16
Description: Allow VPN users to access VPC resources
Authorize Ingress: All users
Access Group ID: (leave empty for all users)
```

#### Allow Internet Access (Optional)
```
Target Network: 0.0.0.0/0
Description: Allow internet access through VPN
Authorize Ingress: All users
```

### Step 4: Associate Subnets and Create Routes

#### Associate with Private Subnets
```
Associate VPN endpoint with:
├── prod-app-1a (10.0.10.0/24)
├── prod-app-1b (10.0.20.0/24)
├── prod-db-1a (10.0.40.0/24)
└── prod-db-1b (10.0.50.0/24)
```

#### Create Route Table Entries
```
Target: 10.0.0.0/16
Origin: add-route
Description: Route to VPC resources
```

---

## Phase 6: Application Load Balancer Setup

### Step 1: Create ALB

#### Basic Configuration
```
Name: prod-app-alb
Scheme: Internet-facing
IP Address Type: IPv4
VPC: prod-vpc
Subnets: prod-public-1a, prod-public-1b, prod-public-1c
Security Groups: prod-alb-sg
```

### Step 2: Configure Target Group

#### Target Group Settings
```
Name: prod-app-tg
Protocol: HTTP
Port: 80
VPC: prod-vpc
Health Check Path: /health
Health Check Interval: 10 seconds
Healthy Threshold: 2
Unhealthy Threshold: 3
```

#### Register Targets
```
Targets: Auto Scaling Group instances
Port: 80 (Nginx proxy port)
```

### Step 3: Configure Listeners

#### HTTP Listener (Redirect to HTTPS)
```
Protocol: HTTP
Port: 80
Action: Redirect to HTTPS
Port: 443
Status Code: 301
```

#### HTTPS Listener
```
Protocol: HTTPS
Port: 443
SSL Certificate: ACM certificate for your domain
Target Group: prod-app-tg
```

---

## Phase 7: Client VPN Access Setup

### Step 1: Download Client Configuration

1. Go to **VPC Console > Client VPN Endpoints**
2. Select your VPN endpoint
3. Click **Download Client Configuration**
4. Save as `prod-client-vpn.ovpn`

### Step 2: Prepare Client Certificates

#### Add certificates to configuration file:
```bash
# Append to prod-client-vpn.ovpn
echo '<cert>' >> prod-client-vpn.ovpn
cat client-cert.pem >> prod-client-vpn.ovpn
echo '</cert>' >> prod-client-vpn.ovpn

echo '<key>' >> prod-client-vpn.ovpn  
cat client-key.pem >> prod-client-vpn.ovpn
echo '</key>' >> prod-client-vpn.ovpn
```

### Step 3: Install VPN Client

#### For Windows/Mac:
1. Download **AWS VPN Client** from AWS website
2. Import `prod-client-vpn.ovpn`
3. Connect using the profile

#### For Linux:
```bash
# Install OpenVPN
sudo apt install openvpn  # Ubuntu/Debian
sudo yum install openvpn  # CentOS/RHEL

# Connect
sudo openvpn --config prod-client-vpn.ovpn
```

---

## Phase 8: Testing and Validation

### Step 1: Test VPN Connection

#### Connect to VPN and verify access:
```bash
# After VPN connection, test connectivity
ping 10.0.10.10  # Application server
ping 10.0.40.10  # Database server

# Test SSH access
ssh -i your-key.pem ec2-user@10.0.10.10  # App server
ssh -i your-key.pem ec2-user@10.0.40.10  # DB server
```

### Step 2: Test Database Access

#### Direct MySQL connection:
```bash
# From your local machine (via VPN)
mysql -h 10.0.40.10 -u appuser -p'AppUserPassword123!' prodapp

# Test connection
mysql> SHOW DATABASES;
mysql> USE prodapp;
mysql> SHOW TABLES;
```

#### Using MySQL Workbench:
```
Connection Name: Production Database
Hostname: 10.0.40.10
Port: 3306
Username: appuser
Password: AppUserPassword123!
```

### Step 3: Test Application Access

#### Internal application testing:
```bash
# Via VPN, access app server directly
curl http://10.0.10.10/health
curl http://10.0.10.10/api/status
```

#### External application testing:
```bash
# Test ALB endpoint
curl https://your-domain.com/health
curl https://your-domain.com/api/status
```

---

## Phase 9: Monitoring and Maintenance

### Step 1: CloudWatch Dashboards

#### Create monitoring dashboard:
```
Dashboard Name: Production Infrastructure
Widgets:
├── ALB Request Count
├── ALB Response Time
├── EC2 CPU Utilization
├── Database Connections
├── VPN Active Connections
└── Error Rates
```

### Step 2: Automated Backups

#### Database Backup Script (Enhanced):
```bash
#!/bin/bash
# Enhanced backup script with error handling

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/tmp"
S3_BUCKET="prod-db-backups"
LOG_FILE="/var/log/backup.log"

# Function to log messages
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# Create backup
log_message "Starting database backup"
if mysqldump -u root -p'YourStrongPassword123!' --single-transaction --routines --triggers --all-databases > $BACKUP_DIR/backup_$DATE.sql; then
    log_message "Database dump successful"
    
    # Compress backup
    gzip $BACKUP_DIR/backup_$DATE.sql
    
    # Upload to S3
    if aws s3 cp $BACKUP_DIR/backup_$DATE.sql.gz s3://$S3_BUCKET/mysql/$(date +%Y)/$(date +%m)/; then
        log_message "Backup uploaded to S3 successfully"
        rm $BACKUP_DIR/backup_$DATE.sql.gz
    else
        log_message "ERROR: Failed to upload backup to S3"
        exit 1
    fi
else
    log_message "ERROR: Database dump failed"
    exit 1
fi

# Cleanup old local backups (optional)
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete

log_message "Backup process completed"
```

### Step 3: Security Monitoring

#### Set up security alerts:
```bash
# Create CloudWatch alarms for security events
aws cloudwatch put-metric-alarm \
  --alarm-name "HighFailedSSHAttempts" \
  --alarm-description "Alert on high failed SSH attempts" \
  --metric-name "FailedSSHAttempts" \
  --namespace "CWLogs" \
  --statistic "Sum" \
  --period 300 \
  --threshold 10 \
  --comparison-operator "GreaterThanThreshold" \
  --evaluation-periods 1
```

---

## Cost Optimization Tips

### 1. Instance Sizing
```
Start Small and Scale:
├── App Servers: t3.medium → scale based on CPU/memory usage
├── DB Servers: t3.medium → upgrade to r5 if memory-intensive
└── Monitor for 30 days before right-sizing
```

### 2. Reserved Instances
```
After 30 days of stable usage:
├── Purchase 1-year Reserved Instances for baseline capacity
├── Use Spot Instances for non-critical app servers
└── Consider Savings Plans for mixed workloads
```

### 3. Storage Optimization
```
Storage Strategy:
├── Use GP3 instead of GP2 (20% cost savings)
├── Implement lifecycle policies for backups
├── Use EBS snapshots for point-in-time recovery
└── Archive old logs to S3 Glacier
```

---

## Security Best Practices Summary

### 1. Network Security
- ✅ All servers in private subnets
- ✅ Security groups with least privilege
- ✅ No direct internet access to database servers
- ✅ VPN access for administrative tasks

### 2. Data Security  
- ✅ Database passwords stored in AWS Secrets Manager (recommended upgrade)
- ✅ Encrypted EBS volumes
- ✅ SSL/TLS for all connections
- ✅ Regular security updates via automated patching

### 3. Access Control
- ✅ IAM roles instead of access keys
- ✅ Certificate-based VPN authentication
- ✅ Multi-factor authentication (recommended)
- ✅ Regular access reviews

This setup provides a production-ready, secure, and scalable infrastructure for your EC2-based application and database servers with convenient Client VPN access for development and administration tasks.
