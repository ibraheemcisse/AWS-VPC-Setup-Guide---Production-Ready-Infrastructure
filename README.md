# AWS-VPC-Setup-Guide---Production-Ready-Infrastructure
AWS VPC setup with NAT Gateway and bastion host architecture using both manual AWS Console method and automated CLI script

# 🚀 AWS VPC Setup Guide - Production-Ready Infrastructure (No NACLs)

> **Enterprise-grade AWS VPC setup with NAT Gateway and bastion host architecture using both manual AWS Console method and automated CLI script**

[![AWS](https://img.shields.io/badge/AWS-VPC-orange?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/vpc/)
[![Bash](https://img.shields.io/badge/Script-Bash-green?logo=gnu-bash&logoColor=white)](https://www.gnu.org/software/bash/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

## 📋 Table of Contents

- [🎯 Project Overview](#-project-overview)
- [🏗️ Architecture](#️-architecture)
- [⚡ Quick Start (Automated)](#-quick-start-automated)
- [📖 Manual Setup Guide](#-manual-setup-guide)
- [🔧 Prerequisites](#-prerequisites)
- [🛠️ Automated Script](#️-automated-script)
- [🔐 Security Configuration](#-security-configuration)
- [❓ Troubleshooting](#-troubleshooting)
- [🧹 Cleanup](#-cleanup)
- [📚 Learning Resources](#-learning-resources)

## 🎯 Project Overview

This repository provides two comprehensive methods to set up a **production-ready AWS VPC infrastructure** with enterprise-grade security and networking components using Security Groups for protection.

### 🌟 What You'll Build:

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| **VPC** | Virtual Private Cloud | `10.0.0.0/16` (65,536 IPs) |
| **Public Subnet** | Internet-facing resources | `10.0.0.0/24` (256 IPs) |
| **Private Subnet** | Secure backend resources | `10.0.1.0/24` (256 IPs) |
| **Internet Gateway** | Public internet access | Attached to VPC |
| **NAT Gateway** | Private subnet internet access | High availability |
| **Route Tables** | Traffic routing | Public & Private |
| **Security Groups** | Instance-level firewall | Least privilege |
| **EC2 Bastion** | Secure SSH access | Public subnet |
| **EC2 Private** | Application server | Private subnet |

### 🎨 Architecture Diagram

```
                          Internet
                             |
                             ▼
    ┌─────────────────────────────────────────────────────────┐
    │                VPC (10.0.0.0/16)                       │
    │                                                         │
    │  ┌─────────────────────┐    ┌─────────────────────┐     │
    │  │   Public Subnet     │    │   Private Subnet    │     │
    │  │    10.0.0.0/24      │    │    10.0.1.0/24      │     │
    │  │                     │    │                     │     │
    │  │ ┌─────────────────┐ │    │ ┌─────────────────┐ │     │
    │  │ │  Bastion Host   │ │    │ │  Private Server │ │     │
    │  │ │   (EC2 Public)  │ │◄──►│ │   (EC2 Private) │ │     │
    │  │ │                 │ │    │ │                 │ │     │
    │  │ └─────────────────┘ │    │ └─────────────────┘ │     │
    │  │                     │    │          │          │     │
    │  │ ┌─────────────────┐ │    │          │          │     │
    │  │ │   NAT Gateway   │◄┼────┼──────────┘          │     │
    │  │ │                 │ │    │                     │     │
    │  │ └─────────────────┘ │    │                     │     │
    │  └─────────────────────┘    └─────────────────────┘     │
    │           │                                             │
    │      ┌─────────┐                                        │
    │      │   IGW   │                                        │
    │      └─────────┘                                        │
    │                                                         │
    │  Security Layer:                                        │
    │  • Security Groups (Instance level)                     │
    └─────────────────────────────────────────────────────────┘
```

### 🔒 Security Features

- **🛡️ Security Groups**: Instance-level firewall protection
- **🔐 Bastion Host**: Secure SSH access to private resources
- **🚫 No Direct Internet**: Private instances access via NAT Gateway
- **📊 Least Privilege**: Minimal required permissions
- **🔍 Network Segmentation**: Isolated subnets

## ⚡ Quick Start (Automated)

### 🚀 One-Command Setup

```bash
# Clone repository
git clone https://github.com/yourusername/aws-vpc-setup-no-nacl.git
cd aws-vpc-setup-no-nacl

# Make script executable
chmod +x setup-vpc-simple.sh

# Run automated setup (NAT Gateway, no NACLs)
./setup-vpc-simple.sh
```

**⏱️ Total Setup Time: ~4 minutes**  
**💰 Cost: ~$35/month (NAT Gateway)**

### 📋 Prerequisites Check

```bash
# Verify AWS CLI
aws --version
aws sts get-caller-identity

# Check key pair exists
aws ec2 describe-key-pairs --key-names YOUR_KEY_NAME

# Verify permissions
aws ec2 describe-vpcs --dry-run
```

## 📖 Manual Setup Guide

### 🏗️ Phase 1: Core Networking

#### Step 1: Create VPC
1. **AWS Console** → **VPC** → **Create VPC**
2. **Settings:**
   ```
   Name: production-vpc
   IPv4 CIDR: 10.0.0.0/16
   IPv6 CIDR: No IPv6
   Tenancy: Default
   ```

#### Step 2: Create Subnets

**Public Subnet:**
```
Name: public-subnet
VPC: production-vpc
AZ: us-east-1a
IPv4 CIDR: 10.0.0.0/24
```

**Private Subnet:**
```
Name: private-subnet  
VPC: production-vpc
AZ: us-east-1a
IPv4 CIDR: 10.0.1.0/24
```

#### Step 3: Internet Gateway
1. **Create Internet Gateway**
   ```
   Name: production-igw
   ```
2. **Attach to VPC:** `production-vpc`

#### Step 4: NAT Gateway
1. **Create NAT Gateway**
   ```
   Name: production-nat
   Subnet: public-subnet (IMPORTANT!)
   Connectivity: Public
   ```
2. **Allocate Elastic IP** (automatically assigned)

### 🛣️ Phase 2: Route Tables

#### Public Route Table
1. **Create Route Table**
   ```
   Name: public-rt
   VPC: production-vpc
   ```
2. **Add Routes:**
   ```
   Destination: 0.0.0.0/0 → Target: production-igw
   ```
3. **Associate:** `public-subnet`

#### Private Route Table  
1. **Create Route Table**
   ```
   Name: private-rt
   VPC: production-vpc
   ```
2. **Add Routes:**
   ```
   Destination: 0.0.0.0/0 → Target: production-nat
   ```
3. **Associate:** `private-subnet`

### 🔒 Phase 3: Security Configuration

#### Security Groups

**Bastion Security Group:**
```bash
Name: bastion-sg
Inbound:
  - SSH (22) from YOUR_IP/32
Outbound:  
  - All traffic to 0.0.0.0/0
```

**Private Security Group:**
```bash
Name: private-sg  
Inbound:
  - SSH (22) from bastion-sg
  - All traffic from 10.0.0.0/16
Outbound:
  - All traffic to 0.0.0.0/0  
```

### 🖥️ Phase 4: EC2 Instances

#### Launch Bastion Host
```bash
AMI: Amazon Linux 2023
Instance Type: t2.micro
Key Pair: your-key-pair
Subnet: public-subnet
Auto-assign Public IP: Enable
Security Group: bastion-sg
```

#### Launch Private Server
```bash
AMI: Amazon Linux 2023  
Instance Type: t2.micro
Key Pair: your-key-pair
Subnet: private-subnet
Auto-assign Public IP: Disable
Security Group: private-sg
```

## 🛠️ Automated Script

```bash
#!/bin/bash

# AWS VPC Simple Setup Script (No NACLs)
# Creates production-ready VPC infrastructure with Security Groups only

set -e  # Exit on any error

# Configuration Variables
VPC_NAME="production-vpc"
KEY_NAME="your-key-name"  # Change this to your key pair name
REGION="us-east-1"
VPC_CIDR="10.0.0.0/16"
PUBLIC_SUBNET_CIDR="10.0.0.0/24"
PRIVATE_SUBNET_CIDR="10.0.1.0/24"
AMI_ID="ami-00ca32bbc84273381"  # Amazon Linux 2023
INSTANCE_TYPE="t2.micro"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
PURPLE='\033[0;35m'
NC='\033[0m' # No Color

echo -e "${PURPLE}🚀 Starting Production VPC Setup (Simplified - No NACLs)...${NC}"

# Function to wait for resource
wait_for_state() {
    local resource_type=$1
    local resource_id=$2
    local desired_state=$3
    
    echo -e "${YELLOW}⏳ Waiting for $resource_type $resource_id to be $desired_state...${NC}"
    aws ec2 wait $resource_type-$desired_state --$resource_type-ids $resource_id
    echo -e "${GREEN}✅ $resource_type is now $desired_state${NC}"
}

# 1. Create VPC
echo -e "${BLUE}📡 Creating VPC...${NC}"
VPC_ID=$(aws ec2 create-vpc \
    --cidr-block $VPC_CIDR \
    --query 'Vpc.VpcId' \
    --output text)

aws ec2 create-tags \
    --resources $VPC_ID \
    --tags Key=Name,Value=$VPC_NAME

aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-support

echo -e "${GREEN}✅ VPC Created: $VPC_ID${NC}"

# 2. Create Internet Gateway
echo -e "${BLUE}🌐 Creating Internet Gateway...${NC}"
IGW_ID=$(aws ec2 create-internet-gateway \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)

aws ec2 create-tags \
    --resources $IGW_ID \
    --tags Key=Name,Value="${VPC_NAME}-igw"

aws ec2 attach-internet-gateway \
    --internet-gateway-id $IGW_ID \
    --vpc-id $VPC_ID

echo -e "${GREEN}✅ Internet Gateway Created: $IGW_ID${NC}"

# 3. Get AZ
AZ=$(aws ec2 describe-availability-zones \
    --query 'AvailabilityZones[0].ZoneName' \
    --output text)

# 4. Create Subnets
echo -e "${BLUE}🏠 Creating Subnets...${NC}"

# Public Subnet
PUBLIC_SUBNET_ID=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block $PUBLIC_SUBNET_CIDR \
    --availability-zone $AZ \
    --query 'Subnet.SubnetId' \
    --output text)

aws ec2 create-tags \
    --resources $PUBLIC_SUBNET_ID \
    --tags Key=Name,Value="${VPC_NAME}-public-subnet"

# Private Subnet  
PRIVATE_SUBNET_ID=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block $PRIVATE_SUBNET_CIDR \
    --availability-zone $AZ \
    --query 'Subnet.SubnetId' \
    --output text)

aws ec2 create-tags \
    --resources $PRIVATE_SUBNET_ID \
    --tags Key=Name,Value="${VPC_NAME}-private-subnet"

echo -e "${GREEN}✅ Subnets Created${NC}"

# 5. Create NAT Gateway
echo -e "${BLUE}🔌 Creating NAT Gateway...${NC}"

# Allocate Elastic IP for NAT Gateway
EIP_ALLOCATION_ID=$(aws ec2 allocate-address \
    --domain vpc \
    --query 'AllocationId' \
    --output text)

aws ec2 create-tags \
    --resources $EIP_ALLOCATION_ID \
    --tags Key=Name,Value="${VPC_NAME}-nat-eip"

# Create NAT Gateway
NAT_GW_ID=$(aws ec2 create-nat-gateway \
    --subnet-id $PUBLIC_SUBNET_ID \
    --allocation-id $EIP_ALLOCATION_ID \
    --query 'NatGateway.NatGatewayId' \
    --output text)

aws ec2 create-tags \
    --resources $NAT_GW_ID \
    --tags Key=Name,Value="${VPC_NAME}-nat-gateway"

# Wait for NAT Gateway to be available
echo -e "${YELLOW}⏳ Waiting for NAT Gateway to be available...${NC}"
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_ID

echo -e "${GREEN}✅ NAT Gateway Created: $NAT_GW_ID${NC}"

# 6. Create Route Tables
echo -e "${BLUE}🛣️  Creating Route Tables...${NC}"

# Public Route Table
PUBLIC_RT_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' \
    --output text)

aws ec2 create-tags \
    --resources $PUBLIC_RT_ID \
    --tags Key=Name,Value="${VPC_NAME}-public-rt"

aws ec2 create-route \
    --route-table-id $PUBLIC_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID

aws ec2 associate-route-table \
    --subnet-id $PUBLIC_SUBNET_ID \
    --route-table-id $PUBLIC_RT_ID

# Private Route Table
PRIVATE_RT_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' \
    --output text)

aws ec2 create-tags \
    --resources $PRIVATE_RT_ID \
    --tags Key=Name,Value="${VPC_NAME}-private-rt"

aws ec2 create-route \
    --route-table-id $PRIVATE_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id $NAT_GW_ID

aws ec2 associate-route-table \
    --subnet-id $PRIVATE_SUBNET_ID \
    --route-table-id $PRIVATE_RT_ID

echo -e "${GREEN}✅ Route Tables Created${NC}"

# 7. Create Security Groups
echo -e "${BLUE}🛡️  Creating Security Groups...${NC}"

# Bastion Security Group
BASTION_SG_ID=$(aws ec2 create-security-group \
    --group-name "${VPC_NAME}-bastion-sg" \
    --description "Security group for bastion host" \
    --vpc-id $VPC_ID \
    --query 'GroupId' \
    --output text)

aws ec2 create-tags \
    --resources $BASTION_SG_ID \
    --tags Key=Name,Value="${VPC_NAME}-bastion-sg"

aws ec2 authorize-security-group-ingress \
    --group-id $BASTION_SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# Private Security Group  
PRIVATE_SG_ID=$(aws ec2 create-security-group \
    --group-name "${VPC_NAME}-private-sg" \
    --description "Security group for private instances" \
    --vpc-id $VPC_ID \
    --query 'GroupId' \
    --output text)

aws ec2 create-tags \
    --resources $PRIVATE_SG_ID \
    --tags Key=Name,Value="${VPC_NAME}-private-sg"

aws ec2 authorize-security-group-ingress \
    --group-id $PRIVATE_SG_ID \
    --protocol tcp \
    --port 22 \
    --source-group $BASTION_SG_ID

aws ec2 authorize-security-group-ingress \
    --group-id $PRIVATE_SG_ID \
    --protocol -1 \
    --cidr $VPC_CIDR

echo -e "${GREEN}✅ Security Groups Created${NC}"

# 8. Launch EC2 Instances
echo -e "${BLUE}🖥️  Launching EC2 Instances...${NC}"

# Launch Bastion Host
BASTION_INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $AMI_ID \
    --count 1 \
    --instance-type $INSTANCE_TYPE \
    --key-name $KEY_NAME \
    --security-group-ids $BASTION_SG_ID \
    --subnet-id $PUBLIC_SUBNET_ID \
    --associate-public-ip-address \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=${VPC_NAME}-bastion-host}]" \
    --query 'Instances[0].InstanceId' \
    --output text)

# Launch Private Instance
PRIVATE_INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $AMI_ID \
    --count 1 \
    --instance-type $INSTANCE_TYPE \
    --key-name $KEY_NAME \
    --security-group-ids $PRIVATE_SG_ID \
    --subnet-id $PRIVATE_SUBNET_ID \
    --no-associate-public-ip-address \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=${VPC_NAME}-private-server}]" \
    --query 'Instances[0].InstanceId' \
    --output text)

echo -e "${GREEN}✅ EC2 Instances Launched${NC}"

# 9. Wait for instances and get details
echo -e "${YELLOW}⏳ Waiting for instances to be running...${NC}"
aws ec2 wait instance-running --instance-ids $BASTION_INSTANCE_ID $PRIVATE_INSTANCE_ID

# Get instance details
BASTION_PUBLIC_IP=$(aws ec2 describe-instances \
    --instance-ids $BASTION_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PublicIpAddress' \
    --output text)

BASTION_PRIVATE_IP=$(aws ec2 describe-instances \
    --instance-ids $BASTION_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PrivateIpAddress' \
    --output text)

PRIVATE_IP=$(aws ec2 describe-instances \
    --instance-ids $PRIVATE_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PrivateIpAddress' \
    --output text)

NAT_PUBLIC_IP=$(aws ec2 describe-addresses \
    --allocation-ids $EIP_ALLOCATION_ID \
    --query 'Addresses[0].PublicIp' \
    --output text)

# 10. Display comprehensive summary
echo -e "\n${PURPLE}🎉 ===== VPC SETUP COMPLETE (NO NACLs)! ===== ${NC}"

echo -e "\n${BLUE}📋 INFRASTRUCTURE SUMMARY:${NC}"
echo -e "┌─────────────────────────────────────────────────┐"
echo -e "│ ${GREEN}VPC Configuration${NC}                            │"
echo -e "├─────────────────────────────────────────────────┤"
echo -e "│ VPC ID: ${YELLOW}$VPC_ID${NC}              │"
echo -e "│ CIDR Block: ${YELLOW}$VPC_CIDR${NC}                      │"
echo -e "│ Internet Gateway: ${YELLOW}$IGW_ID${NC}     │"
echo -e "│ NAT Gateway: ${YELLOW}$NAT_GW_ID${NC}        │"
echo -e "│ NAT Public IP: ${YELLOW}$NAT_PUBLIC_IP${NC}                │"
echo -e "└─────────────────────────────────────────────────┘"

echo -e "\n${BLUE}🏗️ SUBNET CONFIGURATION:${NC}"
echo -e "┌─────────────────────────────────────────────────┐"
echo -e "│ ${GREEN}Public Subnet${NC}                               │"
echo -e "│ ID: ${YELLOW}$PUBLIC_SUBNET_ID${NC}        │"
echo -e "│ CIDR: ${YELLOW}$PUBLIC_SUBNET_CIDR${NC}                    │"
echo -e "│ Route Table: ${YELLOW}$PUBLIC_RT_ID${NC}     │"
echo -e "├─────────────────────────────────────────────────┤"
echo -e "│ ${GREEN}Private Subnet${NC}                              │"
echo -e "│ ID: ${YELLOW}$PRIVATE_SUBNET_ID${NC}       │"
echo -e "│ CIDR: ${YELLOW}$PRIVATE_SUBNET_CIDR${NC}                   │"
echo -e "│ Route Table: ${YELLOW}$PRIVATE_RT_ID${NC}    │"
echo -e "└─────────────────────────────────────────────────┘"

echo -e "\n${BLUE}🖥️ EC2 INSTANCES:${NC}"
echo -e "┌─────────────────────────────────────────────────┐"
echo -e "│ ${GREEN}Bastion Host${NC}                                │"
echo -e "│ Instance ID: ${YELLOW}$BASTION_INSTANCE_ID${NC}    │"
echo -e "│ Public IP: ${YELLOW}$BASTION_PUBLIC_IP${NC}                 │"
echo -e "│ Private IP: ${YELLOW}$BASTION_PRIVATE_IP${NC}                │"
echo -e "│ Security Group: ${YELLOW}$BASTION_SG_ID${NC}     │"
echo -e "├─────────────────────────────────────────────────┤"
echo -e "│ ${GREEN}Private Server${NC}                              │"
echo -e "│ Instance ID: ${YELLOW}$PRIVATE_INSTANCE_ID${NC}    │"
echo -e "│ Private IP: ${YELLOW}$PRIVATE_IP${NC}                      │"
echo -e "│ Security Group: ${YELLOW}$PRIVATE_SG_ID${NC}     │"
echo -e "└─────────────────────────────────────────────────┘"

echo -e "\n${BLUE}🔐 CONNECTION COMMANDS:${NC}"
echo -e "${GREEN}# 1. Connect to Bastion Host:${NC}"
echo -e "ssh -i $KEY_NAME.pem ec2-user@$BASTION_PUBLIC_IP"

echo -e "\n${GREEN}# 2. Copy private key to bastion:${NC}"  
echo -e "scp -i $KEY_NAME.pem $KEY_NAME.pem ec2-user@$BASTION_PUBLIC_IP:~/.ssh/"
echo -e "ssh -i $KEY_NAME.pem ec2-user@$BASTION_PUBLIC_IP 'chmod 400 ~/.ssh/$KEY_NAME.pem'"

echo -e "\n${GREEN}# 3. From bastion, connect to private server:${NC}"
echo -e "ssh -i ~/.ssh/$KEY_NAME.pem ec2-user@$PRIVATE_IP"

echo -e "\n${BLUE}🧪 TESTING CONNECTIVITY:${NC}"
echo -e "${GREEN}# Test internet access from private instance:${NC}"
echo -e "# (Run from private instance)${NC}"
echo -e "curl -I https://www.google.com"
echo -e "# Should work via NAT Gateway!"

echo -e "\n${YELLOW}💰 COST ESTIMATE:${NC}"
echo -e "• NAT Gateway: ~\$32/month"
echo -e "• Elastic IP: ~\$3.6/month"  
echo -e "• EC2 t2.micro: Free tier eligible"
echo -e "• Data transfer: Variable"
echo -e "${RED}Total: ~\$35-40/month${NC}"

echo -e "\n${GREEN}✨ Your simplified VPC infrastructure is live!${NC}"
echo -e "${YELLOW}📚 Security relies entirely on Security Groups.${NC}"
```

## 🔐 Security Configuration

### 🛡️ Simplified Security Strategy

| Security Layer | Purpose | Configuration |
|----------------|---------|---------------|
| **Internet Gateway** | Controlled internet access | Public subnet only |
| **NAT Gateway** | Outbound-only internet for private | High availability, managed |
| **Route Tables** | Traffic direction control | Separate public/private routes |
| **Security Groups** | Instance-level firewall | Stateful, default deny |
| **Bastion Host** | Secure SSH entry point | Jump box architecture |

### 🔒 Security Group Rules

#### Bastion Host Security Group
```bash
# Inbound Rules
Port 22 (SSH): 0.0.0.0/0 or YOUR_IP/32  # Restrict to your IP for production

# Outbound Rules  
All traffic: 0.0.0.0/0  # Allow all outbound
```

#### Private Instance Security Group
```bash
# Inbound Rules
Port 22 (SSH): sg-bastion-id  # Only from bastion security group
All traffic: 10.0.0.0/16      # Internal VPC communication

# Outbound Rules
All traffic: 0.0.0.0/0        # Allow all outbound via NAT
```

## ❓ Troubleshooting

### 🚨 Common Issues & Solutions

#### Connection Issues

**🔧 SSH Connection Refused**

**Problem:** `Connection refused` or `Permission denied`

**Solutions:**
```bash
# 1. Check security group allows SSH (port 22)
aws ec2 describe-security-groups --group-ids sg-xxxxx

# 2. Verify key permissions
chmod 400 your-key.pem

# 3. Check instance is running
aws ec2 describe-instances --instance-ids i-xxxxx

# 4. Verify route table has correct routes
aws ec2 describe-route-tables --route-table-ids rtb-xxxxx
```

**🌐 Private Instance Can't Access Internet**

**Problem:** Private instance can't reach internet via NAT Gateway

**Solutions:**
```bash
# 1. Check NAT Gateway status
aws ec2 describe-nat-gateways --nat-gateway-ids nat-xxxxx

# 2. Verify private route table has NAT route
aws ec2 describe-route-tables --route-table-ids rtb-xxxxx

# 3. Check security group allows outbound traffic
aws ec2 describe-security-groups --group-ids sg-xxxxx

# 4. Test from private instance
curl -I https://www.google.com
```

**💰 High NAT Gateway Costs**

**Problem:** Unexpected NAT Gateway charges

**Solutions:**
- Monitor data transfer in CloudWatch
- Consider VPC endpoints for AWS services
- Use single NAT Gateway for non-critical workloads
- Implement NAT instances for cost optimization

### 📊 Monitoring & Logging

#### Enable VPC Flow Logs
```bash
# Create CloudWatch log group
aws logs create-log-group --log-group-name VPCFlowLogs

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-xxxxx \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name VPCFlowLogs
```

#### Key Metrics to Monitor
- **NAT Gateway**: Data processing charges
- **Elastic IP**: Unused IP addresses  
- **VPC Flow Logs**: Network traffic patterns
- **CloudWatch**: Instance connectivity

## 🧹 Cleanup

### 🗑️ Complete Infrastructure Teardown

```bash
#!/bin/bash
# Simple cleanup script for VPC without NACLs
# Use only when you want to completely remove the infrastructure

set -e

VPC_NAME="production-vpc"
echo "🗑️  Starting infrastructure cleanup for: $VPC_NAME"

# Get VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
    --filters "Name=tag:Name,Values=$VPC_NAME" \
    --query 'Vpcs[0].VpcId' \
    --output text)

if [ "$VPC_ID" = "None" ]; then
    echo "❌ VPC $VPC_NAME not found"
    exit 1
fi

echo "📋 Found VPC: $VPC_ID"

# 1. Terminate EC2 instances
echo "🖥️  Terminating EC2 instances..."
INSTANCE_IDS=$(aws ec2 describe-instances \
    --filters "Name=vpc-id,Values=$VPC_ID" "Name=instance-state-name,Values=running,stopped" \
    --query 'Reservations[].Instances[].InstanceId' \
    --output text)

if [ ! -z "$INSTANCE_IDS" ]; then
    aws ec2 terminate-instances --instance-ids $INSTANCE_IDS
    echo "⏳ Waiting for instances to terminate..."
    aws ec2 wait instance-terminated --instance-ids $INSTANCE_IDS
fi

# 2. Delete NAT Gateways
echo "🔌 Deleting NAT Gateways..."
NAT_GW_IDS=$(aws ec2 describe-nat-gateways \
    --filter "Name=vpc-id,Values=$VPC_ID" \
    --query 'NatGateways[?State==`available`].NatGatewayId' \
    --output text)

if [ ! -z "$NAT_GW_IDS" ]; then
    for nat_id in $NAT_GW_IDS; do
        aws ec2 delete-nat-gateway --nat-gateway-id $nat_id
    done
    echo "⏳ Waiting for NAT Gateways to be deleted..."
    sleep 60  # NAT Gateway deletion takes time
fi

# 3. Release Elastic IPs
echo "📍 Releasing Elastic IPs..."
ALLOCATION_IDS=$(aws ec2 describe-addresses \
    --filters "Name=tag:Name,Values=${VPC_NAME}-nat-eip" \
    --query 'Addresses[].AllocationId' \
    --output text)

if [ ! -z "$ALLOCATION_IDS" ]; then
    for alloc_id in $ALLOCATION_IDS; do
        aws ec2 release-address --allocation-id $alloc_id
    done
fi

# 4. Delete Security Groups (except default)
echo "🛡️  Deleting Security Groups..."
SG_IDS=$(aws ec2 describe-security-groups \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'SecurityGroups[?GroupName!=`default`].GroupId' \
    --output text)

if [ ! -z "$SG_IDS" ]; then
    for sg_id in $SG_IDS; do
        aws ec2 delete-security-group --group-id $sg_id 2>/dev/null || echo "⚠️  Couldn't delete $sg_id (might have dependencies)"
    done
fi

# 5. Delete Route Tables (except main)
echo "🛣️  Deleting Route Tables..."
RT_IDS=$(aws ec2 describe-route-tables \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'RouteTables[?Associations[0].Main!=`true`].RouteTableId' \
    --output text)

if [ ! -z "$RT_IDS" ]; then
    for rt_id in $RT_IDS; do
        aws ec2 delete-route-table --route-table-id $rt_id 2>/dev/null || echo "⚠️  Couldn't delete $rt_id"
    done
fi

# 6. Delete Subnets
echo "🏠 Deleting Subnets..."
SUBNET_IDS=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'Subnets[].SubnetId' \
    --output text)

if [ ! -z "$SUBNET_IDS" ]; then
    for subnet_id in $SUBNET_IDS; do
        aws ec2 delete-subnet --subnet-id $subnet_id
    done
fi

# 7. Detach and Delete Internet Gateway
echo "🌐 Deleting Internet Gateway..."
IGW_ID=$(aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_ID" \
    --query 'InternetGateways[0].InternetGatewayId' \
    --output text)

if [ "$IGW_ID" != "None" ]; then
    aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
    aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
fi

# 8. Delete VPC
echo "📡 Deleting VPC..."
aws ec2 delete-vpc --vpc-id $VPC_ID

echo "✅ Infrastructure cleanup complete!"
echo "💡 Don't forget to check for any remaining resources in the AWS Console"
```

### 💰 Cost Cleanup Checklist

- [ ] **Terminate EC2 instances**
- [ ] **Delete NAT Gateways** (Main cost driver)  
- [ ] **Release Elastic IPs**
- [ ] **Delete unused security groups**
- [ ] **Remove VPC endpoints**
- [ ] **Check CloudWatch logs retention**

## 📚 Learning Resources

### 🎓 AWS Documentation
- [AWS VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)

### 🏗️ Architecture Patterns
- **Multi-AZ Setup**: Extend to multiple availability zones
- **Auto Scaling**: Add load balancers and auto scaling groups  
- **Database Tier**: Add RDS in private subnets
- **CDN Integration**: CloudFront for global content delivery

### 🔧 Advanced Topics
- **VPC Peering**: Connect multiple VPCs
- **Transit Gateway**: Hub-and-spoke networking
- **AWS PrivateLink**: Private connectivity to AWS services
- **Direct Connect**: Dedicated network connection to AWS

### 📊 Monitoring & Optimization
- **VPC Flow Logs**: Network traffic analysis
- **CloudWatch**: Performance monitoring
- **Cost Explorer**: Cost optimization insights
- **AWS Config**: Compliance monitoring

## 🤝 Contributing

### 🔄 How to Contribute

1. **Fork the repository**
2. **Create feature branch**: `git checkout -b feature/amazing-feature`
3. **Commit changes**: `git commit -m 'Add amazing feature'`
4. **Push to branch**: `git push origin feature/amazing-feature`  
5. **Open Pull Request**

### 📝 Contribution Guidelines

- **Documentation**: Update README for any new features
- **Testing**: Test scripts in multiple AWS regions
- **Code Style**: Follow bash best practices
- **Security**: Review security implications
- **Cost**: Consider cost impact of changes

### 🐛 Bug Reports

Use the following template for bug reports:

```markdown
**Environment:**
- AWS Region: 
- AWS CLI Version:
- Operating System:

**Expected Behavior:**
What should happen?

**Actual Behavior:**  
What actually happened?

**Steps to Reproduce:**
1. Step one
2. Step two  
3. Step three

**Additional Context:**
Any relevant logs or screenshots
```

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **AWS Documentation Team** for comprehensive guides
- **AWS Community** for best practices and patterns  
- **Open Source Contributors** for inspiration and feedback

---

### 🌟 Star this repository if it helped you!

**Built with ❤️ for the AWS community**

### 📞 Support

- **AWS Support**: [AWS Support Center](https://support.aws.amazon.com/)

---

> **⚠️ Important**: This infrastructure will incur AWS charges. Always monitor your costs and clean up resources when not needed. The NAT Gateway is the primary cost component at ~$35/month.

## 🔍 Key Differences from NACL Version

### ✅ Advantages of No NACLs:
- **Simplified Setup**: Faster deployment and configuration
- **Easier Management**: One less security layer to maintain
- **Reduced Complexity**: Focus on Security Groups only
- **Lower Learning Curve**: Easier for beginners

### ⚠️ Security Considerations:
- **Single Security Layer**: Only Security Groups provide protection
- **Stateful Only**: No stateless firewall rules
- **Instance-Level Only**: No subnet-level network filtering
- **Default NACL Used**: Uses AWS default permissive NACL rules

### 📋 When to Use This Approach:
- **Development/Testing**: Quick setups for non-production
- **Simple Applications**: Basic web applications
- **Small Teams**: Limited security expertise
- **Learning AWS**: Understanding VPC basics first

### 🛡️ Security Best Practices:
- **Restrict Bastion Access**: Use your specific IP instead of 0.0.0.0/0
- **Regular Security Reviews**: Audit Security Group rules periodically
- **VPC Flow Logs**: Enable for network monitoring
- **CloudTrail**: Enable for API call logging
- **AWS Config**: Monitor compliance and configuration changes
