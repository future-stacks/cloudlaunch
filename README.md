# CloudLaunch Platform

A lightweight cloud platform demonstrating secure website hosting and document storage using AWS core services (S3, IAM, VPC).

## 🏗️ Architecture Overview

This project implements a secure, scalable architecture with:
- **Static Website Hosting**: Public S3 bucket with CloudFront distribution
- **Private Document Storage**: Secure S3 buckets with restricted access
- **Network Segmentation**: 3-tier VPC architecture (Public, Application, Database)
- **Identity & Access Management**: Granular permissions using IAM policies

```
Internet
    ↓
[CloudFront CDN] → [S3 Website Bucket]
    ↓
[Internet Gateway]
    ↓
┌─────────────────────────────────────────────────┐
│                CloudLaunch VPC                  │
│                 (10.0.0.0/16)                   │
│                                                 │
│  Public Subnet (10.0.1.0/24)                   │
│  ├── Load Balancers (future)                    │
│                                                 │
│  App Subnet (10.0.2.0/24) - Private            │
│  ├── Application Servers (future)               │
│                                                 │
│  DB Subnet (10.0.3.0/28) - Private             │
│  ├── Database Services (future)                 │
└─────────────────────────────────────────────────┘

[Private S3 Buckets]
├── cloudlaunch-private-bucket (Read/Write Access)
└── cloudlaunch-visible-only-bucket (List Only)
```

## 🚀 Live Demo

- **Website URL**: http://cloudlaunch-site-bucket-20250823.s3-website-eu-west-1.amazonaws.com/
- **CloudFront URL**: http://d3ubpddfy195jy.cloudfront.net/

## 📋 Task 1: S3 Static Website Hosting & IAM

### S3 Buckets Created

#### 1. `cloudlaunch-site-bucket-20250823`
- **Purpose**: Hosts the public static website
- **Access**: Public read access for anonymous users
- **Features**: 
  - Static website hosting enabled
  - Custom error pages configured
  - CloudFront distribution (optional)

#### 2. `cloudlaunch-private-bucket-20250824`
- **Purpose**: Private document storage
- **Access**: Limited to IAM user with GetObject/PutObject permissions
- **Security**: No public access, no delete permissions

#### 3. `cloudlaunch-visible-only-bucket-20250824`
- **Purpose**: Demonstration of list-only permissions
- **Access**: IAM user can see the bucket exists but cannot access contents
- **Security**: No public access, list permissions only

### IAM Policy Implementation

The `cloudlaunch-user` has been configured with the following custom policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket-20250823",
                "arn:aws:s3:::cloudlaunch-private-bucket-20250824",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket-20250824"
            ]
        },
        {
            "Sid": "GetObjectSiteBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket-20250823/*"
        },
        {
            "Sid": "GetPutObjectPrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-private-bucket-20250824/*"
        },
        {
            "Sid": "VPCReadOnlyAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInternetGateways"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:Region": "Europe (Ireland) eu-west-1"
                }
            }
        }
    ]
}
```

### Key Policy Features
- **Least Privilege**: User can only access specified buckets
- **No Delete Permissions**: Prevents accidental data loss
- **Granular Access**: Different permissions for different buckets
- **VPC Read Access**: Can view but not modify VPC resources

## 🌐 Task 2: VPC Network Architecture

### VPC Configuration

**VPC Name**: `cloudlaunch-vpc`  
**CIDR Block**: `10.0.0.0/16`  
**Region**: `Europe (Ireland) eu-west-1`

### Subnet Design

#### Public Subnet
- **Name**: `cloudlaunch-public-subnet`
- **CIDR**: `10.0.1.0/24`
- **Purpose**: Load balancers and public-facing services
- **Internet Access**: ✅ Yes (via Internet Gateway)
- **Route Table**: `cloudlaunch-public-rt`

#### Application Subnet (Private)
- **Name**: `cloudlaunch-app-subnet`
- **CIDR**: `10.0.2.0/24`
- **Purpose**: Application servers and business logic
- **Internet Access**: ❌ No (private subnet)
- **Route Table**: `cloudlaunch-app-rt`

#### Database Subnet (Private)
- **Name**: `cloudlaunch-db-subnet`
- **CIDR**: `10.0.3.0/28` (16 IP addresses)
- **Purpose**: Database services (RDS, etc.)
- **Internet Access**: ❌ No (private subnet)
- **Route Table**: `cloudlaunch-db-rt`

### Network Routing

#### Public Route Table (`cloudlaunch-public-rt`)
```
Destination     | Target
10.0.0.0/16     | local
0.0.0.0/0       | igw-xxxxxxxxx (Internet Gateway)
```

#### Private Route Tables (`cloudlaunch-app-rt`, `cloudlaunch-db-rt`)
```
Destination     | Target
10.0.0.0/16     | local
```
*Note: No internet route (0.0.0.0/0) ensures these subnets remain private*

### Security Groups

#### Application Security Group (`cloudlaunch-app-sg`)
```
Type    | Protocol | Port | Source
Inbound | HTTP     | 80   | 10.0.0.0/16 (VPC only)
```

#### Database Security Group (`cloudlaunch-db-sg`)
```
Type    | Protocol | Port | Source
Inbound | MySQL    | 3306 | 10.0.2.0/24 (App subnet only)
```

### Network Security Features
- **Defense in Depth**: Multiple layers of security controls
- **Network Segmentation**: Isolated tiers for different application components
- **Least Privilege Access**: Security groups allow only necessary traffic
- **Private Subnets**: Database and application tiers have no direct internet access

## 🔐 Access Information

### AWS Account Details
- **Account ID**: `862287594288`
- **Account Alias**: `cloudlaunchdemo`
- **Console URL**: `https://cloudlaunchdemo.signin.aws.amazon.com/console`

### IAM User Credentials
- **Username**: `cloudlaunch-user`
- **Access Key ID**: `[Provided separately for security]`
- **Secret Access Key**: `[Provided separately for security]`
- **Password Policy**: Force password change on first login ✅
- **MFA Required**: Recommended for production use

⚠️ **Security Note**: Credentials are provided separately. Change the password immediately upon first login.

## 🧪 Testing & Verification

### S3 Functionality Tests

```bash
# Configure AWS CLI with cloudlaunch-user credentials
aws configure

# Test 1: List all accessible buckets (should work)
aws s3 ls

# Test 2: Access website bucket (should work)
aws s3 cp s3://cloudlaunch-site-bucket-20250823/index.html ./test-download.html

# Test 3: Upload to private bucket (should work)
echo "Test file" > test.txt
aws s3 cp test.txt s3://cloudlaunch-private-bucket-20250824/

# Test 4: Try to delete from private bucket (should fail - no permission)
aws s3 rm s3://cloudlaunch-private-bucket-20250824/test.txt

# Test 5: List visible-only bucket (should work)
aws s3 ls s3://cloudlaunch-visible-only-bucket-20250824/

# Test 6: Try to access visible-only bucket contents (should fail)
aws s3 cp s3://cloudlaunch-visible-only-bucket-20250824/anyfile.txt ./
```

### VPC Verification

1. **Subnet Associations**: Verify each subnet is associated with correct route table
2. **Security Groups**: Confirm rules allow only intended traffic
3. **Internet Gateway**: Attached only to VPC, routed only to public subnet
4. **Private Subnets**: No routes to 0.0.0.0/0 (confirmed internet isolation)

## 🎯 Key Learning Outcomes

### AWS Services Mastered
- **Amazon S3**: Static website hosting, bucket policies, access control
- **AWS IAM**: Custom policies, least privilege access, user management
- **Amazon VPC**: Network design, subnets, routing, security groups

### Best Practices Implemented
- **Security**: Multi-layered security with IAM, bucket policies, and network ACLs
- **Cost Optimization**: Free tier usage only, no unnecessary resources
- **Documentation**: Comprehensive documentation and clear architecture diagrams
- **Scalability**: Design allows for future expansion with additional services

### Real-World Applications
- **Web Application Hosting**: Scalable static site with CDN
- **Document Management**: Secure file storage with granular access
- **Network Security**: Enterprise-grade network segmentation
- **Identity Management**: Role-based access control systems

## 🛠️ Technologies Used

- **AWS S3**: Object storage and static website hosting
- **AWS CloudFront**: Content delivery network (CDN)
- **AWS IAM**: Identity and access management
- **AWS VPC**: Virtual private cloud networking
- **HTML/CSS**: Frontend website development
- **JSON**: Policy document formatting

## 🚀 Future Enhancements

- [ ] Add SSL/TLS certificates for custom domain
- [ ] Implement AWS Lambda for dynamic functionality
- [ ] Add Amazon RDS for database services
- [ ] Set up AWS CloudWatch for monitoring and logging
- [ ] Implement AWS WAF for web application firewall
- [ ] Add backup and disaster recovery procedures

## 📞 Support

For technical support or questions about this implementation:
- Review AWS documentation: https://docs.aws.amazon.com
- Contact AWS Support: https://support.aws.amazon.com
- Check CloudFormation templates for infrastructure as code

## 📄 License

This project is for educational purposes as part of the AltSchool Cloud Engineering curriculum.

---

**Project Completed**: 24th August 2025 

**Author**: Hope Akpabio

**Course**: AltSchool of Engineering - Cloud Engineering Track 

**Assignment**: Third Semester Assignment 1
