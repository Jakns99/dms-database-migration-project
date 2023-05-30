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
  - The first step was to set up the lab environment using CloudFormation. The lab generously provides a couple of resources to start off the project, such as a starter VPC without routes as to let me create them, a couple of EC2 instances, and a few password and username credentials. Base lab infrastructure was provided to allow for focused efforts on learning.
