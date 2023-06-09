# Database Migration Project
This project showcases an advanced lab demo of migrating a simple web application (Wordpress) from an on-premises environment into AWS using the AWS Database Migration Service (DMS). The on-premises environment consists of a virtual web server and a MariaDB database server, both simulated using EC2 servers. The target environment in AWS consists of an EC2 web server and an RDS managed SQL database, there is an included VPC for practice in networking. 

This hands-on project provides valuable insights into the process of database migration using AWS DMS and demonstrates the practical application of AWS services in managing and migrating databases. The experience gathered from this project highlights essential concepts and skills necessary for handling database migration taks in real-world scenarios.
## Disclaimer/Project Reference
**This project was completed with the instructions from https://github.com/acantril/learn-cantrill-io-labs, specifically the aws-dms-database-migration lab. This project was done as a learning experience and I take no credit from the source. With that being said, I added my own explanations to why each section and technology were used and how they all fit together to showcase what I've learned.**

## Architecture Overview

![Architecture](DMS_architecture.png)

- **This architecture image above is based on the architecture image provided by https://github.com/acantril/learn-cantrill-io-labs**

This Architecture/Project consists of five stages, each employing key AWS services:

1. Provision the environment and review tasks (CloudFormation)
2. Establish Private Connectivity Between the environments (VPC Peering)
3. Create and Configure the AWS Infrastructure (EC2 and DB)
4. Migrate Database and Cutover (DMS)
5. Cleanup

## Key Features
- **Database Migration (DMS)**: Leveraging AWS Database Migration Service(DMS), the project effectively migrates a MariaDB database from a simulated on-premises environment to an AWS managed SQL database in RDS. This service simplifies the migration process, mitigating risks and minimizing downtime.
- **Private Connectivity (VPC)**: To maintain a secure and reliable connection, the project establishes a Virtual Private Cloud (VPC) peering connection between the simulated on-premises and AWS environments. This ensures seamless, secure networking and efficient data transfer during the data migration process.
- **Managed AWS Services (EC2/RDS)**: By using managed AWS services like EC2 for web hosting and RDS for database management, the project demonstrates how to offload server management tasks to AWS, enabling users to focus more on application development.


# Project Setup/How I Completed This Project
 This project was established and configured manually utilizing the AWS Management Console with a provided CloudFormation template. Here's a step-by-step outline of the setup process:
 
 ## 1. Provision Environment and Tasks
  - The first step was to set up the lab environment using a CloudFormation stack. The lab generously provides a couple of resources to start off the project, such as a starter VPC without routes as to let me create them, a couple of EC2 instances, and a few password and username credentials. Base lab infrastructure was provided to allow for focused efforts on learning.
  
  ![Cloudformation](/Data%20Migration/Cloudformation.png)

 ## 2A. Create VPC Peering Connections
  - This step had me creating a VPC peering connection between the On-Premises and AWS environments. 
  - The process of peering is very simple, and it involves provisioning a peering connection via VPC's peering connection tab. The requester was onpremVPC and the Accepter was awsVPC. Next, I accepted the request to confirm the peering connection.
  
  ![VPC Peering](/Data%20Migration/Peering%20vpc.png)

## 2B. Create Routes
- This step involved delving into the Routes of my new public and private route tables. This involved adding the IP 192.168.10.0/24 Peering Connection to my Public Route Table and 192.168.10.0/24 to my Private table.

![Routes](/Data%20Migration/Route%20creation%20example.png)

**Explanation for Step 2**: By creating these peering connections and routes, I am effectively setting up the VPC for secure and seamless connectivity between the On-Premises and AWS environments. The peering connection allows for direct network routing between the two VPCs, enabling instances in each VPC to communicate with each other as if they were within the same network. Meanwhile, the new routes ensure that traffic destined for the 192.168.10.0/24 IP range is correctly directed via the peering connection. This setup forms the basis for the efficient and secure data transfer that is crucial for the subsequent database migration process.

## 3A. Create the RDS Instance
 - This step had me provisioning an RDS DB Subnet Group and a MariaDB Database instance.
 - A DB subnet group is a collection of subnets that you designate for your DB instances in the VPC, and allows the RDS instance to use multiple AZ.

![DB Subnet Groups](/Data%20Migration/DB%20Subnet%20Groups.png)


 - The MariaDB instance was configured with the settings provided by the project, such as instance specs, storage capacity, security groups etc. The setup for the RDS instance normally takes a long time, so I moved on to the next steps while it was provisioning.

## 3B + 3C. Create the EC2 Instance and Install Wordpress
 - This step involved the creation of an instance named awsCatWeb. This step included the choosing of an AMI, which was Linux 2 AMI (HVM), the VPC I edited earlier, an IAM Instance Profile, and choosing to move on without a key pair.
 - Once the instance was ready, I connected to it via Session Manager and ran a sudo bash command to run a privileged bash shell. Once a few commands were run, I ran `yum -y install httpd mariadb` to install the apache web server, and then `amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2 ` to install php.
 - At this point, I now had a functioning apache web server. 
 - The details behind the cat website below were created by the original project creators. I provisioned it for learning purposes.

![CatWeb](/Data%20Migration/CATWEB%20instance.png)

## 3D + 3E. Migrate Wordpress Content over and Fix up Permissions and Verify Server Functionality
 - In this step I ran a `nano /etc/ssh/sshd_config` edit to allow password authentication with the CloudFormation stack passwords.

![Nano](/Data%20Migration/Nano%20example.png)

 - Once this was complete, I connected to the stack provisioned CatWeb instance with Session Manager, with a `sudo bash` command, I ran a provided command to copy the wordpress files over from CatWeb to awsCatWeb, which was essentially transferring on-prem data to aws.

![WordpressMigrate](/Data%20Migration/Migrating%20wordpress%20content%20over.png)
- This snapshot above is a picture I took while it was running.

 - Lastly, I ran this provided command
 ```
usermod -a -G apache ec2-user   
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
sudo systemctl restart httpd
```
- This script was used to  configure permissions and ownership for the web server in a Linux environment. It added the ec2-user to the apache group, changed the ownership of the "/var/www" directory to ec2-user and apache, and set directory and file permissions to ensure that both the user and group can read, write and execute files. Lastly, it restarts the httpd service to apply these changes.

**Explanation for Step 3**: This entire step is fundamental in building the AWS side of the infrastructure for the database migration process. It began by setting up an RDS instance that will host the migrated MariaDB database. Once the database setup was in progress, an EC2 instance was created to serve as the new webserver for the WordPress application. This involved the installation of an Apache web server and PHP, setting up the foundation for the application layer of the architecture. Following this, the WordPress content was migrated from the on-premises environment to the newly created AWS environment. This migration essentially transfers the WordPress application files from the on-premises instance (CatWeb) to the new AWS EC2 instance (awsCatWeb).

Finally, permissions and ownership were configured on the AWS web server to ensure secure and efficient access to the WordPress files. The Apache group was given the necessary permissions to the '/var/www' directory and its contents, ensuring that the web server can correctly serve the WordPress application.


## 4A. Create the DMS Subnet Group
- Firstly, I created a DMS subnet group, which is a collection of subnets that DMS will use when creating the replication instance

## 4B. Create DMS Replication Instance
- Then, I created a DMS replication instance.
- Replication instances perform the actual data migration tasks, including extracting data from the source database, formatting the data for consumption by the target database, and loading the data into the target database.

![Replication](/Data%20Migration/DMS%20Replication%20Instance.png)

## 4C and 4D. Create the DMS Source Endpoint and Destination Endpoint (RDS)
- Following this, I then set up the DMS Source and Destination (target) endpoints.
- The Source Endpoint pointed to the MariaDB on the on-prem EC2 instance (CatDB), while the target endpoint points to the MariaDB instance on RDS (a4lwordpress).

![DMS Endpoints](/Data%20Migration/DMS%20Endpoints.png)

## 4E and 4F. Test the Endpoints and Migrate
- After setting up the endpoints, I tested the connection between them to ensure that the DMS replication instance can successfully access both the source and target databases.
- Next, I created a migration taks to transfer the database data from the source to the target endpoint. This task specified what data is to be migrated, how it should be migrated, and where it should be migrated to.

![MigStart](/Data%20Migration/DMS%20Migration%20Start.png)

![MigEnd](/Data%20Migration/DMS%20Complete.png)
## 4G Cutover Application Instance
- Finally, after the data migration task completed and the data was successfully loaded into the RDS instance, the final cutover step involved updating the WordPress application to point to the new RDS database instance.
- This was done by updating the 'DB_HOST' line in the WordPress wp-config.php file to point to the RDS instance's endpoint. A script was also run to update the WordPress database with the new AWS instance DNS name. It is listed below.
```
#!/bin/bash
source <(php -r 'require("/var/www/html/wp-config.php"); echo("DB_NAME=".DB_NAME."; DB_USER=".DB_USER."; DB_PASSWORD=".DB_PASSWORD."; DB_HOST=".DB_HOST); ')
SQL_COMMAND="mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e"
OLD_URL=$(mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e 'select option_value from wp_options where option_id = 1;' | grep http)
HOST=$(curl http://169.254.169.254/latest/meta-data/public-hostname)
$SQL_COMMAND "UPDATE wp_options SET option_value = replace(option_value, '$OLD_URL', 'http://$HOST') WHERE option_name = 'home' OR option_name = 'siteurl';"
$SQL_COMMAND "UPDATE wp_posts SET guid = replace(guid, '$OLD_URL','http://$HOST');"
$SQL_COMMAND "UPDATE wp_posts SET post_content = replace(post_content, '$OLD_URL', 'http://$HOST');"
$SQL_COMMAND "UPDATE wp_postmeta SET meta_value = replace(meta_value,'$OLD_URL','http://$HOST');"

```
- This step completed the migration of the application to the AWS environment, making the AWS RDS instance the new live database.

**Explanation for Step 4**: This was the centerpiece of the project after all of the setup. This step focused on setting up AWS Database Migration Service (DMS) to migrate the database from the on-premises environment to the AWS environment. AWS DMS conducted a seamless database transition, reducing downtime during the migration process. Ultimately, the aim was to streamline the transition to the AWS environment, thus creating a scalable foundation for future work on the system. This project effectively simulated a transition from an on-prem database to AWS via EC2, since I had no access to a real on-prem database.

## 5. Cleanup
 - The easiest step, I first cleaned up the EC2 instances (CatWeb and CatDB) by stopping them. I terminated awsCatWeb
 - For RDS, I terminated the RDS instance created earlier, unchecking all boxes to prevent further costs due to backup storage.
 - Next, DMS termination was simple, I deleted the migration task, the replication instances, the endpoints, and the subnet group
 - Deleting the VPC involved manually checking each route table and deleting the peer routes and peering connection created earlier. This concludes the deletion of the manually created resources
 - Lastly, I deleted the DMS Stack in CloudFormation, effectively removing all provisioned resources for the project.


