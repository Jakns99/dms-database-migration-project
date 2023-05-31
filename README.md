# Database Migration Project
This project showcases an advanced lab demo of migrating a simple web application (Wordpress) from an on-premises environment into AWS using the AWS Database Migration Service (DMS). The on-premises environment consists of a virtual web server and a MariaDB database server, both simulated using EC2 servers. The target environment in AWS consists of an EC2 web server and an RDS managed SQL database, there is an included VPC for practice in networking. 

This hands-on project provides valuable insights into the process of database migration using AWS DMS and demonstrates the practical application of AWS services in managing and migrating databases. The experience gathered from this project highlights essential concepts and skills necessary for handling database migration taks in real-world scenarios.
## Disclaimer/Project Reference
This project is not mine nor is it my idea. This project was completed with the instructions from https://github.com/acantril/learn-cantrill-io-labs, specifically the aws-dms-database-migration lab. This project was done as a learning experience and I take no credit from the source. With that being said, I added my own explanations to why each section and technology were used and how they all fit together to showcase what I've learned.

## Architecture Overview


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

 ## 2A. Create VPC Peering Connections
  - This step had me creating a VPC peering connection between the On-Premises and AWS environments. 
  - The process of peering is very simple, and it involves provisioning a peering connection via VPC's peering connection tab. The requester was onpremVPC and the Accepter was awsVPC. Next, I accepted the request to confirm the peering connection.

## 2B. Create Routes
- This step involved delving into the Routes of my new public and private route tables. This involved adding the IP 192.168.10.0/24 Peering Connection to my Public Route Table and 192.168.10.0/24 to my Private table.

**Explanation for Step 2**: By creating these peering connections and routes, I am effectively setting up the VPC for secure and seamless connectivity between the On-Premises and AWS environments. The peering connection allows for direct network routing between the two VPCs, enabling instances in each VPC to communicate with each other as if they were within the same network. Meanwhile, the new routes ensure that traffic destined for the 192.168.10.0/24 IP range is correctly directed via the peering connection. This setup forms the basis for the efficient and secure data transfer that is crucial for the subsequent database migration process.

## 3A. Create the RDS Instance
 - This step had me provisioning an RDS DB Subnet Group and a MariaDB Database instance.
 - A DB subnet group is a collection of subnets that you designate for your DB instances in the VPC, and allows the RDS instance to use multiple AZ.
 - The MariaDB instance was configured with the settings provided by the project, such as instance specs, storage capacity, security groups etc. The setup for the RDS instance normally takes a long time, so I moved on to the next steps while it was provisioning.

## 3B + 3C. Create the EC2 Instance and Install Wordpress
 - This step involved the creation of an instance named awsCatWeb. This step included the choosing of an AMI, which was Linux 2 AMI (HVM), the VPC I edited earlier, an IAM Instance Profile, and choosing to move on without a key pair.
 - Once the instance was ready, I connected to it via Session Manager and ran a sudo bash command to run a privileged bash shell. Once a few commands were run, I ran `yum -y install httpd mariadb` to install the apache web server, and then `amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2 ` to install php.
 - At this point, I now had a functioning apache web server.

## 3D + 3E. Migrate Wordpress Content over and Fix up Permissions and Verify Server Functionality
 - In this step I ran a `nano /etc/ssh/sshd_config` edit to allow password authentication with the CloudFormation stack passwords.
 - Once this was complete, I connected to the stack provisioned CatWeb instance with Session Manager, with a `sudo bash` command, I ran a provided command to copy the wordpress files over from CatWeb to awsCatWeb, which was essentially transferring on-prem data to aws.
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
## 4C and 4D. Create the DMS Source Endpoint and Destination Endpoint (RDS)
- Following this, I then set up the DMS Source and Destination (target) endpoints.
- The Source Endpoint pointed to the MariaDB on the on-prem EC2 instance (CatDB), while the target endpoint points to the MariaDB instance on RDS (a4lwordpress).
## 4E and 4F. Test the Endpoints and Migrate
- After setting up the endpoints, I tested the connection between them to ensure that the DMS replication instance can successfully access both the source and target databases.
- Next, I created a migration taks to transfer the database data from the source to the target endpoint. This task specified what data is to be migrated, how it should be migrated, and where it should be migrated to.
## 4G Cutover Application Instance










