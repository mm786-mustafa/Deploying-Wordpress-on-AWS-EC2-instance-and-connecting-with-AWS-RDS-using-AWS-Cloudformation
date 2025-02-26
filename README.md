# Deploying WordPress on AWS using CloudFormation

## Overview
This CloudFormation template automates the deployment of a WordPress website on an EC2 instance, connected to an RDS MySQL database. It also provisions networking resources such as a VPC, subnets, security groups, and an Application Load Balancer (ALB) for high availability and security.

---
## **Table of Contents**
- [Parameters](#parameters)
- [Mappings](#mappings)
- [Resources](#resources)
- [User Data Script](#user-data-script)
- [Outputs](#outputs)

---

## **Parameters**

The template defines several parameters that allow customization of the deployment.

### **Environment Settings**
- **Environment**: Defines the deployment environment (`dev`, `testing`, or `production`).
- **KeyName**: The name of an existing EC2 key pair to SSH into the instance.

### **Networking Parameters**
- **VpcName**: The name of the VPC being created.
- **VpcCIDR**: The IP range for the VPC.
- **InternetGatewayName**: Name of the Internet Gateway.
- **NatGatewayName**: Name of the NAT Gateway for private subnets.
- **PublicRouteTableName**: Name of the route table for public subnets.
- **PrivateRouteTableName**: Name of the route table for private subnets.
- **Subnets**: List of subnets to be created.
- **AvailabilityZones**: AWS availability zones for high availability.

### **Security and Access**
- **SecurityGroupName**: Defines a security group allowing HTTP and MySQL access.
- **MySqlCidr**: CIDR block allowed for MySQL access.
- **HttpAccessCidr**: CIDR block allowed for HTTP access.

### **Database Parameters**
- **RdsInstanceName**: Name of the RDS instance.
- **DatabaseName**: Name of the database.
- **DatabaseUsername**: Username for database authentication.
- **DatabaseUserPassword**: Password for database authentication.

---
## **Mappings**
Mappings are used to set instance types and AMIs based on the selected environment.

### **EnvironmentConfig Mapping**
Maps environments to specific instance types and AMIs:
- `dev`: Uses `t2.micro` and an AMI for development.
- `testing`: Uses `t2.micro` with a different AMI for testing.
- `production`: Uses `t2.micro` for production workloads.

### **CidrIndexPublicSubnets & CidrIndexPrivateSubnets**
These mappings define indexes for CIDR blocks assigned to public and private subnets in different availability zones.

---
## **Resources**
This section outlines the infrastructure components created by the template.

### **VPC and Networking**
- `MyVPC`: Creates a custom VPC with the specified CIDR block.
- `MyInternetGateway`: Creates an Internet Gateway and attaches it to the VPC.
- `AttachVpc`: Associates the Internet Gateway with the VPC.
- `ElasticIPAddress`: Creates an Elastic IP for the NAT Gateway.
- `MyNatGateway`: Creates a NAT Gateway for private subnet internet access.

### **Subnets and Routing**
Public and private subnets are dynamically created using `Fn::ForEach`.
- `MyPublicSubnet&{AZ}`: Creates public subnets in specified availability zones.
- `MyPrivateSubnet&{AZ}`: Creates private subnets.
- `MyPublicRouteTable` and `MyPrivateRouteTable`: Define separate route tables for public and private traffic.
- `PublicRoute1`: Routes public subnet traffic to the Internet Gateway.
- `PrivateRoute1`: Routes private subnet traffic through the NAT Gateway.

### **Security Group**
- `MySecurityGroup`: Allows inbound HTTP (port 80) and MySQL (port 3306) traffic.

### **Database (RDS)**
- `MyRdsDatabase`: Provisions an Amazon RDS MySQL instance.
- `MyDBSubnetGroup`: Defines the subnet group for RDS within private subnets.

### **EC2 Instance**
- `MyEC2Instance`: Provisions an EC2 instance with WordPress installed.
- Uses `UserData` to install Apache, MySQL, PHP, and WordPress automatically.

### **Application Load Balancer (ALB)**
- `MyTargetGroup`: Registers the EC2 instance as a target.
- `MyALB`: Provisions an internet-facing Application Load Balancer.
- `MyALBListener`: Routes traffic to the target group.

---
## **User Data Script**
The EC2 instance is configured with a User Data script to install WordPress:
```bash
#!/bin/bash
sudo yum update -y
sudo amazon-linux-extras install php7.4 -y
sudo yum install httpd -y
sudo yum install mysql -y
sudo yum install php-mysqlnd php-fpm php-json php-xml php-gd php-mbstring -y
sudo systemctl enable httpd
sudo systemctl start httpd
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo mv wordpress/* /var/www/html/
sudo chown -R apache:apache /var/www/html/
sudo systemctl restart httpd
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo sed -i "s/database_name_here/${DatabaseName}/" /var/www/html/wp-config.php
sudo sed -i "s/username_here/${DatabaseUsername}/" /var/www/html/wp-config.php
sudo sed -i "s/password_here/${DatabaseUserPassword}/" /var/www/html/wp-config.php
sudo sed -i "s/localhost/${MyRdsDatabase.Endpoint.Address}/" /var/www/html/wp-config.php
sudo systemctl restart httpd
```
---
## **Deployment Steps**
1. Save the CloudFormation template as `wordpress-deployment.yaml`.
2. Navigate to the AWS CloudFormation Console.
3. Click **Create Stack** and upload the template.
4. Provide necessary parameters (KeyPair, Subnets, AZs, etc.).
5. Click **Create Stack** and wait for deployment.
6. Once completed, find the ALB DNS name in the Outputs section.
7. Open the ALB URL in a browser to access WordPress.

---
## **Conclusion**
This CloudFormation template simplifies the deployment of WordPress on AWS using EC2 and RDS, ensuring a scalable, secure, and high-availability setup. Modify parameters as needed to suit different environments.

---

