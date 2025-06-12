
# wordpress-webapp-commands


![2 _Host_a_WordPress_Website_on_AWS](https://github.com/user-attachments/assets/848f3b39-9e4f-49f6-a63b-6ba93797bf85)


Deploying a WordPress Site on AWS

This repository contains the configuration files and setup scripts used to launch a WordPress website on Amazon Web Services (AWS). It utilizes a combination of AWS services to provide a highly available, scalable, and secure hosting environment for the WordPress application.

Architecture Overview

The WordPress application runs on Amazon EC2 instances within a secure and resilient infrastructure that features:

A Virtual Private Cloud (VPC) configured with public and private subnets across two Availability Zones (AZs) to support redundancy and high availability.
An Internet Gateway for enabling communication between the VPC and the internet.
Security Groups that function as virtual firewalls to manage access to resources.
Public Subnets for the NAT Gateway and Application Load Balancer, which handle external requests and internet-bound traffic.
Private Subnets dedicated to web server instances for added security.
EC2 Instance Connect Endpoint to allow secure, browser-based SSH access.
An Application Load Balancer linked with a target group to distribute traffic evenly across EC2 instances.
An Auto Scaling Group that dynamically increases or decreases the number of EC2 instances based on load.
Amazon RDS to provide a reliable, managed database backend.
Amazon EFS for shared and scalable file storage between instances.
AWS Certificate Manager (ACM) to provision and manage SSL/TLS certificates.
Amazon SNS to send notifications regarding Auto Scaling events.
Amazon Route 53 for DNS management and custom domain routing.
Deployment Scripts

WordPress Setup Script
This script initializes an EC2 instance for WordPress, including Apache, PHP, MySQL installation, and EFS mounting:

# Switch to the root user
sudo su

# Update package repositories and installed packages
sudo yum update -y

# Create the target directory for the website
sudo mkdir -p /var/www/html

# Define the EFS DNS name
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount EFS to the web directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache HTTP server and enable it to run on boot
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP and required extensions
sudo dnf install -y \
php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd \
php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath \
php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Download and install MySQL 8 community release
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Configure file and directory permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
sudo chown -R apache:apache /var/www/html

# Download and extract WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Prepare wp-config.php
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo vi /var/www/html/wp-config.php

# Restart Apache to apply changes
sudo service httpd restart
Auto Scaling Launch Template Script
This script is embedded into the launch template so that all newly launched EC2 instances in the Auto Scaling Group are properly initialized.

#!/bin/bash

# Update all system packages
sudo yum update -y

# Install Apache and enable it at boot
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP and required modules
sudo dnf install -y \
php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd \
php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath \
php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Set up MySQL repository and install MySQL server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable MySQL service
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Mount EFS to web directory using fstab
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Apply proper permissions
chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
Getting Started

To use this project:

Clone the repository to your local system.
Set up the AWS infrastructure components (VPC, subnets, NAT Gateway, etc.) as described in the architecture overview.
Run the installation script to configure an EC2 instance with WordPress.
Set up the Auto Scaling Group and Application Load Balancer to manage scaling and traffic.
Access your WordPress site through the Load Balancer's DNS endpoint.
Contributing

Feel free to contribute by submitting pull requests. Fork the repository, make your changes, and propose your enhancements.

License

This project is licensed under the MIT License. See the LICENSE file for more details.

