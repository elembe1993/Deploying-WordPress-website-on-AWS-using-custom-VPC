# Deploying-WordPress-website-on-AWS-using-custom-VPC
Hosting a WordPress Website on AWS
This repository contains the resources and scripts used to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application. Below is a detailed guide on the architecture, deployment scripts, and steps to set up the WordPress site.

Architecture Overview
The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

Virtual Private Cloud (VPC): Configured with public and private subnets across two Availability Zones for fault tolerance and high availability.
Internet Gateway: Facilitates connectivity between VPC instances and the internet.
Security Groups: Acts as a virtual firewall to control inbound and outbound traffic.
Public Subnets: Used for infrastructure components like the NAT Gateway and Application Load Balancer.
Private Subnets: Used for web servers to enhance security.
EC2 Instance Connect Endpoint: Provides secure connections to instances within both public and private subnets.
Application Load Balancer: Distributes incoming web traffic across multiple EC2 instances.
Auto Scaling Group: Automatically adjusts the number of EC2 instances based on traffic.
Amazon RDS: Managed relational database service for the WordPress database.
Amazon EFS: Scalable, elastic file storage system for shared file storage.
AWS Certificate Manager: Manages SSL/TLS certificates for securing application communications.
AWS SNS: Provides notifications related to Auto Scaling Group activities.
Amazon Route 53: For domain name registration and DNS management.
Deployment Scripts
WordPress Installation Script
This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, and mounting the Amazon EFS to the instance.

bash
Copy code
# Switch to root user
sudo su

# Update the software packages on the EC2 instance 
sudo yum update -y

# Create an html directory 
sudo mkdir -p /var/www/html

# Environment variable for EFS DNS name
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount the EFS to the html directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install the Apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 along with several necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
sudo chown apache:apache -R /var/www/html 

# Download WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit the wp-config.php file (modify database connection details as needed)
sudo vi /var/www/html/wp-config.php

# Restart the web server
sudo service httpd restart
Auto Scaling Group Launch Template Script
This script is included in the launch template for the Auto Scaling Group, ensuring that new instances are configured correctly with the necessary software and settings.

bash
Copy code
#!/bin/bash
# Update the software packages on the EC2 instance 
sudo yum update -y

# Install the Apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 along with several necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Environment variable for EFS DNS name
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# Mount the EFS to the html directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
sudo chown apache:apache -R /var/www/html

# Restart the web server
sudo service httpd restart
How to Use
Clone this repository to your local machine.
Follow the AWS documentation to create the required resources (VPC, subnets, Internet Gateway, etc.) as outlined in the architecture overview.
Use the provided scripts to set up the WordPress application on EC2 instances within the VPC.
Configure the Auto Scaling Group, Load Balancer, and other services as per the architecture.
Access the WordPress website through the Load Balancer's DNS name.
References
AWS VPC Documentation
Amazon EC2 User Guide
AWS Auto Scaling Guide
Amazon RDS Documentation
Amazon EFS Documentation
AWS Certificate Manager
Amazon Route 53 Documentation
Contributing
Contributions to this project are welcome! Please fork the repository and submit a pull request with your enhancements.
