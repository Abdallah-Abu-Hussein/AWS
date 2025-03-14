# **AWS Infrastructure Setup Report**

[Video]()

## **1. Introduction**
This report outlines the step-by-step process of setting up a cloud-based infrastructure on AWS using the AWS Command Line Interface (CLI). The infrastructure includes a Virtual Private Cloud (VPC), subnets, an Internet Gateway, NAT Gateway, Security Groups, EC2 instances, an Elastic Load Balancer (ELB), and an RDS MySQL database.

## **2. Project Objectives**
The goal of this project is to:
- Implement a secure and scalable AWS network infrastructure.
- Automate resource creation using AWS CLI.
- Deploy a web application and database in a private subnet.
- Enable internet access via a NAT Gateway and load balancing using an ELB.

## **3. Infrastructure Design**
The architecture consists of:
- **VPC:** A private network in AWS.
- **Subnets:** Public and private subnets for organizing resources.
- **Internet Gateway (IGW):** To provide internet access for public resources.
- **NAT Gateway:** Allows outbound internet access for private resources.
- **Security Groups:** To control inbound and outbound traffic.
- **Elastic Load Balancer (ELB):** For distributing traffic to multiple instances.
- **EC2 Instances:** Web server in the private subnet.
- **RDS MySQL Database:** Hosted in a private subnet for backend storage.

## **4. Step-by-Step Implementation**

### **4.1. Install and Configure AWS CLI**
1. Install AWS CLI on Arch Linux:
   ```sh
   sudo yay -S aws-cli
   ```
2. Verify installation:
   ```sh
   aws --version
   ```
3. Configure AWS CLI:
   ```sh
   aws configure
   ```
   - Enter AWS Access Key ID, Secret Access Key, Default Region, and Output Format.

### **4.2. Create a Virtual Private Cloud (VPC)**
1. Create a VPC with CIDR block `10.0.0.0/16`:
   ```sh
   aws ec2 create-vpc --cidr-block 10.0.0.0/16
   ```
2. Enable DNS support and hostnames:
   ```sh
   aws ec2 modify-vpc-attribute --vpc-id VPC_ID --enable-dns-support
   aws ec2 modify-vpc-attribute --vpc-id VPC_ID --enable-dns-hostnames
   ```

### **4.3. Create Subnets**
1. **DMZ Public Subnet (for ELB, NAT, SSH)**:
   ```sh
   aws ec2 create-subnet --vpc-id VPC_ID --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
   ```
2. **Front-end Private Subnet (Web Server)**:
   ```sh
   aws ec2 create-subnet --vpc-id VPC_ID --cidr-block 10.0.2.0/24 --availability-zone us-east-1b
   ```
3. **Back-end Private Subnet (Database)**:
   ```sh
   aws ec2 create-subnet --vpc-id VPC_ID --cidr-block 10.0.3.0/24 --availability-zone us-east-1c
   ```

### **4.4. Create an Internet Gateway (IGW) and Attach It to VPC**
1. Create an Internet Gateway:
   ```sh
   aws ec2 create-internet-gateway
   ```
2. Attach IGW to the VPC:
   ```sh
   aws ec2 attach-internet-gateway --vpc-id VPC_ID --internet-gateway-id IGW_ID
   ```

### **4.5. Set Up Route Tables**
1. Create a route table for the public subnet:
   ```sh
   aws ec2 create-route-table --vpc-id VPC_ID
   ```
2. Add a default route to the Internet Gateway:
   ```sh
   aws ec2 create-route --route-table-id RTB_ID --destination-cidr-block 0.0.0.0/0 --gateway-id IGW_ID
   ```
3. Associate the Route Table with the Public Subnet:
   ```sh
   aws ec2 associate-route-table --subnet-id DMZ_SUBNET_ID --route-table-id RTB_ID
   ```

### **4.6. Security Group Configuration**
1. Allow SSH (port 22) and HTTP (port 8080) for the Web Server:
   ```sh
   aws ec2 authorize-security-group-ingress --group-id SG_ID --protocol tcp --port 8080 --cidr 0.0.0.0/0
   ```
2. Allow MySQL access (port 3306) only from the Web Server:
   ```sh
   aws ec2 authorize-security-group-ingress --group-id MYSQL_SG_ID --protocol tcp --port 3306 --source-group WEBAPP_SG_ID
   ```

### **4.7. Deploy EC2 Instances**
1. **Web Server (Private Subnet):**
   ```sh
   aws ec2 run-instances --image-id ami-12345678 --count 1 --instance-type t2.micro --key-name MyKey --security-group-ids SG_ID --subnet-id FRONTEND_SUBNET_ID
   ```
2. **MySQL Database (Private Subnet):**
   ```sh
   aws ec2 run-instances --image-id ami-12345678 --count 1 --instance-type t2.micro --key-name MyKey --security-group-ids MYSQL_SG_ID --subnet-id BACKEND_SUBNET_ID
   ```

### **4.8. Deploy Load Balancer and NAT Gateway**
1. **Create Elastic Load Balancer (ELB):**
   ```sh
   aws elb create-load-balancer --load-balancer-name MyELB --listeners Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=8080 --subnets DMZ_SUBNET_ID --security-groups ELB_SG_ID
   ```
2. **Create NAT Gateway for private instances:**
   ```sh
   aws ec2 create-nat-gateway --subnet-id DMZ_SUBNET_ID --allocation-id ALLOCATION_ID
   ```




