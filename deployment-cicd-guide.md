# TaskFlow Pro - AWS EC2 Deployment & CI/CD Pipeline Guide

## Table of Contents
1. [Overview](#overview)
2. [AWS Infrastructure Setup](#aws-infrastructure-setup)
3. [EC2 Instance Configuration](#ec2-instance-configuration)
4. [Application Deployment](#application-deployment)
5. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
6. [Monitoring & Logging](#monitoring-logging)
7. [Security Best Practices](#security-best-practices)
8. [Backup & Disaster Recovery](#backup-disaster-recovery)
9. [Troubleshooting](#troubleshooting)

---

## 1. OVERVIEW

### 1.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        Users/Clients                         │
└────────────────────────────┬────────────────────────────────┘
                             │ HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    Route 53 (DNS)                            │
│              taskflowpro.example.com                         │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│            Application Load Balancer (ALB)                   │
│              SSL/TLS Termination                             │
│              Health Checks                                   │
└────────────────────────────┬────────────────────────────────┘
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
┌───────────────────────┐    ┌───────────────────────┐
│   EC2 Instance (App)  │    │   EC2 Instance (App)  │
│   - Nginx             │    │   - Nginx             │
│   - FastAPI (Uvicorn) │    │   - FastAPI (Uvicorn) │
│   - Angular (Static)  │    │   - Angular (Static)  │
│   - Redis             │    │   - Redis             │
│   - Celery Workers    │    │   - Celery Workers    │
└───────────────────────┘    └───────────────────────┘
                │                         │
                └────────────┬────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    RDS PostgreSQL                            │
│              - Multi-AZ Deployment                           │
│              - Automated Backups                             │
│              - Read Replicas (Optional)                      │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    S3 Bucket                                 │
│              - File Attachments                              │
│              - Static Assets                                 │
│              - Backups                                       │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Environment Architecture

```
Production Environment:
├── 2x EC2 t3.large (App Servers) - us-east-1a, us-east-1b
├── 1x RDS db.t3.medium (PostgreSQL) - Multi-AZ
├── 1x ALB (Application Load Balancer)
├── 1x S3 Bucket (Files & Backups)
├── ElastiCache Redis (Optional)
└── CloudWatch (Monitoring)

Staging Environment:
├── 1x EC2 t3.medium (App Server)
├── 1x RDS db.t3.small (PostgreSQL)
└── S3 Bucket (Files)
```

---

## 2. AWS INFRASTRUCTURE SETUP

### 2.1 Prerequisites

- AWS Account with admin access
- AWS CLI installed and configured
- Domain name (for SSL certificate)
- GitHub account for CI/CD

### 2.2 VPC & Network Setup

#### Create VPC
```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=TaskFlowPro-VPC}]'

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-xxxxx \
  --enable-dns-hostnames
```

#### Create Subnets
```bash
# Public Subnet 1 (us-east-1a)
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=TaskFlowPro-Public-1a}]'

# Public Subnet 2 (us-east-1b)
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=TaskFlowPro-Public-1b}]'

# Private Subnet 1 (us-east-1a) - For RDS
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.11.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=TaskFlowPro-Private-1a}]'

# Private Subnet 2 (us-east-1b) - For RDS
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.12.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=TaskFlowPro-Private-1b}]'
```

#### Create Internet Gateway
```bash
# Create IGW
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=TaskFlowPro-IGW}]'

# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxxxx \
  --vpc-id vpc-xxxxx
```

#### Configure Route Tables
```bash
# Create route table for public subnets
aws ec2 create-route-table \
  --vpc-id vpc-xxxxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=TaskFlowPro-Public-RT}]'

# Add route to IGW
aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxxxx

# Associate with public subnets
aws ec2 associate-route-table \
  --route-table-id rtb-xxxxx \
  --subnet-id subnet-xxxxx
```

### 2.3 Security Groups

#### Application Server Security Group
```bash
# Create security group
aws ec2 create-security-group \
  --group-name TaskFlowPro-App-SG \
  --description "Security group for TaskFlowPro application servers" \
  --vpc-id vpc-xxxxx

# Allow SSH (from specific IP - replace with your IP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 22 \
  --cidr YOUR_IP/32

# Allow HTTP from ALB
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 80 \
  --source-group sg-alb-xxxxx

# Allow HTTPS from ALB
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 443 \
  --source-group sg-alb-xxxxx

# Allow FastAPI port from ALB
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 8000 \
  --source-group sg-alb-xxxxx
```

#### ALB Security Group
```bash
# Create ALB security group
aws ec2 create-security-group \
  --group-name TaskFlowPro-ALB-SG \
  --description "Security group for TaskFlowPro ALB" \
  --vpc-id vpc-xxxxx

# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-alb-xxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-alb-xxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

#### RDS Security Group
```bash
# Create RDS security group
aws ec2 create-security-group \
  --group-name TaskFlowPro-RDS-SG \
  --description "Security group for TaskFlowPro RDS" \
  --vpc-id vpc-xxxxx

# Allow PostgreSQL from app servers
aws ec2 authorize-security-group-ingress \
  --group-id sg-rds-xxxxx \
  --protocol tcp \
  --port 5432 \
  --source-group sg-xxxxx
```

### 2.4 RDS PostgreSQL Setup

#### Create DB Subnet Group
```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name taskflowpro-db-subnet \
  --db-subnet-group-description "Subnet group for TaskFlowPro RDS" \
  --subnet-ids subnet-private-1a subnet-private-1b \
  --tags Key=Name,Value=TaskFlowPro-DB-Subnet
```

#### Create RDS Instance
```bash
aws rds create-db-instance \
  --db-instance-identifier taskflowpro-db-prod \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --engine-version 15.4 \
  --master-username dbadmin \
  --master-user-password 'YourStrongPassword123!' \
  --allocated-storage 100 \
  --storage-type gp3 \
  --storage-encrypted \
  --vpc-security-group-ids sg-rds-xxxxx \
  --db-subnet-group-name taskflowpro-db-subnet \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "mon:04:00-mon:05:00" \
  --multi-az \
  --publicly-accessible false \
  --tags Key=Name,Value=TaskFlowPro-DB-Prod
```

**Note the endpoint**: `taskflowpro-db-prod.xxxxx.us-east-1.rds.amazonaws.com`

### 2.5 S3 Bucket Setup

```bash
# Create S3 bucket
aws s3api create-bucket \
  --bucket taskflowpro-files-prod \
  --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket taskflowpro-files-prod \
  --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket taskflowpro-files-prod \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'

# Set lifecycle policy (optional - delete old versions after 90 days)
aws s3api put-bucket-lifecycle-configuration \
  --bucket taskflowpro-files-prod \
  --lifecycle-configuration file://s3-lifecycle.json
```

**s3-lifecycle.json**:
```json
{
  "Rules": [
    {
      "Id": "DeleteOldVersions",
      "Status": "Enabled",
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 90
      }
    }
  ]
}
```

### 2.6 IAM Roles & Policies

#### EC2 Instance Role
```bash
# Create trust policy
cat > ec2-trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create role
aws iam create-role \
  --role-name TaskFlowPro-EC2-Role \
  --assume-role-policy-document file://ec2-trust-policy.json

# Create policy for S3 access
cat > s3-access-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::taskflowpro-files-prod",
        "arn:aws:s3:::taskflowpro-files-prod/*"
      ]
    }
  ]
}
EOF

# Attach policy
aws iam put-role-policy \
  --role-name TaskFlowPro-EC2-Role \
  --policy-name S3Access \
  --policy-document file://s3-access-policy.json

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name TaskFlowPro-EC2-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name TaskFlowPro-EC2-Profile \
  --role-name TaskFlowPro-EC2-Role
```

---

## 3. EC2 INSTANCE CONFIGURATION

### 3.1 Launch EC2 Instances

#### Create Launch Template
```bash
# Create user data script
cat > user-data.sh << 'EOF'
#!/bin/bash
# Update system
yum update -y

# Install Docker
amazon-linux-extras install docker -y
systemctl start docker
systemctl enable docker
usermod -a -G docker ec2-user

# Install Docker Compose
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Install Git
yum install git -y

# Install CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm

# Create application directory
mkdir -p /opt/taskflowpro
chown ec2-user:ec2-user /opt/taskflowpro
EOF

# Create launch template
aws ec2 create-launch-template \
  --launch-template-name TaskFlowPro-App-Template \
  --version-description "Production app server template" \
  --launch-template-data '{
    "ImageId": "ami-0c55b159cbfafe1f0",
    "InstanceType": "t3.large",
    "IamInstanceProfile": {
      "Name": "TaskFlowPro-EC2-Profile"
    },
    "SecurityGroupIds": ["sg-xxxxx"],
    "UserData": "'$(base64 -w 0 user-data.sh)'",
    "TagSpecifications": [{
      "ResourceType": "instance",
      "Tags": [
        {"Key": "Name", "Value": "TaskFlowPro-App"},
        {"Key": "Environment", "Value": "Production"}
      ]
    }],
    "BlockDeviceMappings": [{
      "DeviceName": "/dev/xvda",
      "Ebs": {
        "VolumeSize": 50,
        "VolumeType": "gp3",
        "DeleteOnTermination": true,
        "Encrypted": true
      }
    }]
  }'
```

#### Launch Instances
```bash
# Launch instance 1 (us-east-1a)
aws ec2 run-instances \
  --launch-template LaunchTemplateName=TaskFlowPro-App-Template \
  --subnet-id subnet-public-1a \
  --count 1 \
  --key-name your-key-pair

# Launch instance 2 (us-east-1b)
aws ec2 run-instances \
  --launch-template LaunchTemplateName=TaskFlowPro-App-Template \
  --subnet-id subnet-public-1b \
  --count 1 \
  --key-name your-key-pair
```

### 3.2 Initial Server Setup

**SSH into each instance**:
```bash
ssh -i your-key.pem ec2-user@ec2-xx-xx-xx-xx.compute.amazonaws.com
```

#### Install Additional Dependencies
```bash
# Install Python 3.11
sudo yum install python3.11 python3.11-pip -y

# Install Nginx
sudo amazon-linux-extras install nginx1 -y

# Install Node.js (for Angular build)
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install nodejs -y

# Verify installations
docker --version
docker-compose --version
python3.11 --version
nginx -v
node --version
npm --version
```

### 3.3 Application Directory Structure

```bash
# Create directory structure
sudo mkdir -p /opt/taskflowpro/{backend,frontend,nginx,logs,data}
sudo chown -R ec2-user:ec2-user /opt/taskflowpro

# Navigate to app directory
cd /opt/taskflowpro
```

### 3.4 Environment Configuration

#### Create Environment File
```bash
# Create .env file
cat > /opt/taskflowpro/.env << EOF
# Application
APP_NAME=TaskFlowPro
APP_ENV=production
DEBUG=false
SECRET_KEY=your-super-secret-key-change-this
API_VERSION=v1

# Database
DATABASE_URL=postgresql://dbadmin:YourStrongPassword123!@taskflowpro-db-prod.xxxxx.us-east-1.rds.amazonaws.com:5432/taskflowpro
DB_POOL_SIZE=20
DB_MAX_OVERFLOW=10

# Redis
REDIS_URL=redis://localhost:6379/0
REDIS_CACHE_DB=1

# JWT
JWT_SECRET_KEY=your-jwt-secret-key-change-this
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=30
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# Email
SMTP_HOST=email-smtp.us-east-1.amazonaws.com
SMTP_PORT=587
SMTP_USER=your-ses-user
SMTP_PASSWORD=your-ses-password
SMTP_FROM=noreply@taskflowpro.com

# AWS S3
AWS_REGION=us-east-1
S3_BUCKET=taskflowpro-files-prod
S3_ACCESS_KEY=your-access-key
S3_SECRET_KEY=your-secret-key

# Celery
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# CORS
CORS_ORIGINS=["https://taskflowpro.com","https://www.taskflowpro.com"]

# Monitoring
SENTRY_DSN=your-sentry-dsn
LOG_LEVEL=INFO
EOF

# Secure the file
chmod 600 /opt/taskflowpro/.env
```

### 3.5 Docker Compose Configuration

```bash
cat > /opt/taskflowpro/docker-compose.yml << EOF
version: '3.8'

services:
  backend:
    image: taskflowpro/backend:latest
    container_name: taskflowpro-backend
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "8000:8000"
    volumes:
      - ./logs/backend:/app/logs
    depends_on:
      - redis
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  celery-worker:
    image: taskflowpro/backend:latest
    container_name: taskflowpro-celery-worker
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - ./logs/celery:/app/logs
    depends_on:
      - redis
      - backend
    command: celery -A app.celery_worker worker --loglevel=info --concurrency=4

  celery-beat:
    image: taskflowpro/backend:latest
    container_name: taskflowpro-celery-beat
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - ./logs/celery:/app/logs
    depends_on:
      - redis
      - backend
    command: celery -A app.celery_worker beat --loglevel=info

  redis:
    image: redis:7-alpine
    container_name: taskflowpro-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - ./data/redis:/data
    command: redis-server --appendonly yes --maxmemory 512mb --maxmemory-policy allkeys-lru
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

networks:
  default:
    name: taskflowpro-network
EOF
```

### 3.6 Nginx Configuration

```bash
# Create Nginx config
sudo cat > /etc/nginx/conf.d/taskflowpro.conf << 'EOF'
# Rate limiting
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=auth_limit:10m rate=5r/m;

# Upstream backend
upstream backend {
    least_conn;
    server localhost:8000 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

# HTTP to HTTPS redirect
server {
    listen 80;
    server_name taskflowpro.com www.taskflowpro.com;
    
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
    
    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name taskflowpro.com www.taskflowpro.com;
    
    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/taskflowpro.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/taskflowpro.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Logging
    access_log /var/log/nginx/taskflowpro-access.log;
    error_log /var/log/nginx/taskflowpro-error.log;
    
    # Client body size
    client_max_body_size 50M;
    
    # Frontend (Angular)
    location / {
        root /opt/taskflowpro/frontend/dist;
        try_files $uri $uri/ /index.html;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # API endpoints
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        proxy_buffering off;
        proxy_request_buffering off;
    }
    
    # Authentication endpoints (stricter rate limit)
    location ~ ^/api/v1/(auth|login|register) {
        limit_req zone=auth_limit burst=5 nodelay;
        
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # WebSocket endpoint
    location /ws {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_read_timeout 86400;
    }
    
    # Health check endpoint (no logging)
    location /health {
        access_log off;
        proxy_pass http://backend;
    }
    
    # Static files with caching
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        root /opt/taskflowpro/frontend/dist;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
EOF

# Test Nginx configuration
sudo nginx -t

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 3.7 SSL Certificate (Let's Encrypt)

```bash
# Install Certbot
sudo yum install certbot python3-certbot-nginx -y

# Obtain certificate
sudo certbot --nginx -d taskflowpro.com -d www.taskflowpro.com \
  --non-interactive --agree-tos --email admin@taskflowpro.com

# Auto-renewal (already set up by certbot)
sudo certbot renew --dry-run
```

---

## 4. APPLICATION DEPLOYMENT

### 4.1 Manual Deployment (First Time)

```bash
# Clone repository
cd /opt/taskflowpro
git clone https://github.com/your-org/taskflowpro.git repo
cd repo

# Backend setup
cd backend
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run database migrations
export DATABASE_URL="postgresql://dbadmin:password@rds-endpoint:5432/taskflowpro"
alembic upgrade head

# Build backend Docker image
docker build -t taskflowpro/backend:latest .

# Frontend build
cd ../frontend
npm install
npm run build:prod

# Copy build to nginx directory
sudo cp -r dist/* /opt/taskflowpro/frontend/dist/

# Start services
cd /opt/taskflowpro
docker-compose up -d

# Check status
docker-compose ps
docker-compose logs -f
```

### 4.2 Deployment Script

Create deployment script for future use:

```bash
cat > /opt/taskflowpro/deploy.sh << 'EOF'
#!/bin/bash

set -e

echo "=== TaskFlowPro Deployment Script ==="
echo "Starting deployment at $(date)"

# Variables
REPO_DIR="/opt/taskflowpro/repo"
APP_DIR="/opt/taskflowpro"
BACKUP_DIR="/opt/taskflowpro/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Create backup
echo "Creating backup..."
mkdir -p $BACKUP_DIR
docker-compose exec -T backend pg_dump $DATABASE_URL > $BACKUP_DIR/db_backup_$TIMESTAMP.sql

# Pull latest code
echo "Pulling latest code..."
cd $REPO_DIR
git fetch origin
git checkout main
git pull origin main

# Backend deployment
echo "Building backend..."
cd $REPO_DIR/backend
docker build -t taskflowpro/backend:$TIMESTAMP .
docker tag taskflowpro/backend:$TIMESTAMP taskflowpro/backend:latest

# Run migrations
echo "Running database migrations..."
docker-compose run --rm backend alembic upgrade head

# Frontend deployment
echo "Building frontend..."
cd $REPO_DIR/frontend
npm install
npm run build:prod

# Copy to nginx directory
echo "Deploying frontend..."
sudo rm -rf $APP_DIR/frontend/dist/*
sudo cp -r dist/* $APP_DIR/frontend/dist/

# Restart services
echo "Restarting services..."
cd $APP_DIR
docker-compose up -d --force-recreate

# Health check
echo "Performing health check..."
sleep 10
curl -f http://localhost:8000/health || exit 1

# Cleanup old images
echo "Cleaning up old Docker images..."
docker image prune -af --filter "until=72h"

echo "=== Deployment completed successfully at