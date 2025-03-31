# AWS VPC Setup Guide

## Overview
This provides a step-by-step process for setting up a **Virtual Private Cloud (VPC)** in AWS and configuring essential components such as **Subnets, Route Tables, Internet Gateway, NAT Gateway**, and launching an **EC2 instance with Tomcat as a Jump Server**.

![Image Alt](https://github.com/Lisaeileen/AWS-VPC-setup/blob/967953be70ec1091d219b1a989de30cbab48f99e/vpc-architecture.png)

## Prerequisites
- An **AWS Account**
- AWS **IAM user** with required permissions
- Basic understanding of **AWS networking**
- AWS CLI (optional for automation)

---

## **Step 1: Create a Virtual Private Cloud (VPC)**
1. Log in to **AWS Management Console**.
2. Navigate to **VPC Dashboard** → Click **Create VPC**.
3. Enter the following details:
   - **Name:** `vpc-project`
   - **IPv4 CIDR Block:** `10.0.0.0/24`
   - **IPv6 CIDR Block:** None (optional)
   - **Tenancy:** Default
4. Click **Create VPC**.

![Image Alt](https://github.com/Lisaeileen/AWS-VPC-setup/blob/ed426a0c2159868466dfa9aebb61c82bf85ce4ec/vpc.png)

---

## **Step 2: Create Subnets**
### **Public Subnet**
1. Navigate to **Subnets** → Click **Create Subnet**.
2. Choose **vpc-project**.
3. Enter details:
   - **Name:** `Public-sn`
   - **CIDR Block:** `10.0.1.0/25`
   - **Availability Zone:** Choose any. I used us-east-1a
4. Click **Create Subnet**.
5. Enable **Auto-assign Public IP** for the subnet.

### **Private Subnet**
1. Click **Create Subnet**.
2. Choose **vpc-project**.
3. Enter details:
   - **Name:** `Private-sn`
   - **CIDR Block:** `10.0.0.128/25`
   - **Availability Zone:** Same or different from PublicSubnet. I used us-east-1b
4. Click **Create Subnet**.

![Image Alt](https://github.com/Lisaeileen/AWS-VPC-setup/blob/b50b3c3523fdb8c63a9a05a820c8184b9a7810f9/subnet.png)

---

## **Step 3: Create an Internet Gateway (IGW)**
1. Navigate to **Internet Gateways** → Click **Create Internet Gateway**.
2. Enter **Name:** `igw-c16`.
3. Click **Create** and then **Attach** it to `vpc-project`.

![Image Alt](https://github.com/Lisaeileen/AWS-VPC-setup/blob/b50b3c3523fdb8c63a9a05a820c8184b9a7810f9/igw.png)

---

## **Step 4: Create a Route Table**
### **Public Route Table**
1. Navigate to **Route Tables** → Click **Create Route Table**.
2. Enter:
   - **Name:** `Public-rt`
   - **VPC:** `vpc-project`
3. Click **Create**.
4. Edit the Route Table:
   - Add a new route:
     - **Destination:** `0.0.0.0/0`
     - **Target:** `igw-c16`
5. Associate this route table with `Public-sn`.

### **Private Route Table**
1. Create another **Route Table** named `Private-rt`.
2. Associate it with `Private-sn` (without Internet access).

---

## **Step 5: Create a NAT Gateway**
1. Navigate to **NAT Gateway** → Click **Create NAT Gateway**.
2. Select **Public-sn**.
3. Allocate an **Elastic IP**.
4. Click **Create NAT Gateway**.
5. **Update Private Route Table**:
   - **Destination:** `0.0.0.0/0`
   - **Target:** `NAT Gateway`

---

## **Step 6: Launch Tomcat JumpServer EC2 Instance**
1. Navigate to **EC2 Dashboard** → Click **Launch Instance**.
2. Choose an **Amazon Linux Image (AMI)**:
   - `Amazon Linux 2` 
3. Choose an **Instance Type**:
   - `t2.micro` (Free Tier) 
4. Configure instance:
   - **VPC:** `vpc-project`
   - **Subnet:** `Public-sn`
   - **Auto-assign Public IP:** Enabled
   - **Security Group:** Create one allowing:
     - **SSH (22):** Your IP
     - **HTTP (8080):** Custom TCP
5. Add **User Data** to install Tomcat:
   ```bash
   #!/bin/bash
   sudo yum update
   sudo yum install java -y
   sudo wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.5/bin/apache-tomcat-11.0.5.tar.gz
   sudo tar -xzf apache-tomcat-11.0.5.tar.gz
   sudo rm apache-tomcat-11.0.5.tar.gz
   sudo mkdir /opt/tomcat11
   sudo mv apache-tomcat-11.0.5/* /opt/tomcat11
   sudo chmod 777 -R /opt/tomcat11
   sudo sh /opt/tomcat11/bin/startup.sh
   sudo ln -s /opt/tomcat11/bin/startup.sh /usr/bin/starttomcat
   sudo ln -s /opt/tomcat11/bin/shutdown.sh /usr/bin/stoptomcat
   sudo starttomcat
   ```
6. **Launch Instance** and copy the **Public IP**.

---

## **Step 7: Access Tomcat Server**
1. Open a browser and enter:
   ```
   http://<EC2-Public-IP>:8080
   ```
2. You should see the **Tomcat Welcome Page**.

![Image Alt](https://github.com/Lisaeileen/AWS-VPC-setup/blob/b50b3c3523fdb8c63a9a05a820c8184b9a7810f9/tomcat.png)

---

## **Conclusion**
This setup provides:
- **Secure AWS VPC** with public and private subnets.
- **Internet Gateway & NAT Gateway** for controlled internet access.
- **JumpServer EC2 Instance** running **Apache Tomcat**.

This architecture can be further expanded to host **multi-tier applications** securely.

### **Next Steps**
- Deploy an **Auto Scaling Group** for high availability.
- Integrate **RDS (Relational Database Service)** for backend storage.
- Implement **CloudWatch Monitoring & Alerts**.

---



