# VPC with public-private subnet in production

![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/8f45e87c-0abd-483d-8a45-6d7c69829af4)

Image source: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html

## Overview
- The VPC has public subnets and private subnets in two Availability Zones.
- Each public subnet contains a NAT gateway and a load balancer node.
- The servers run in the private subnets, are launched and terminated by using an Auto Scaling group, and receive traffic from the load balancer.
- The servers can connect to the internet by using the NAT gateway.

This demonstrates how to create a VPC that you can use for servers in a production environment. To improve resiliency (your organization's ability to handle failure while remaining functional), servers are deployed in two AZs by using an Auto Scaling group and an Application Load Balancer. For additional security, servers are deployed in private subnets. The servers receive requests through the load balancer. The servers can connect to the internet by using a NAT gateway. Also, the NAT gateway is deployed in both AZs.

## Step 1: Create VPC

1. Log in to the AWS Management Console.
2. In the AWS Console, search for `VPC`.
3. Within the VPC dashboard, select the option to create a VPC: `Create VPC`.
4. Choose `VPC and more` option to simplify configuration (Other wise we have to create the Subnets, IPv4, IPv6 and other necessary configurations).
5. Name the VPC as `aws-prod-example` (Name tag auto-generation).
6. Ensure that there are two public and two private subnets.
7. Set up 1 NAT Gateway per Availability Zone: `1 per AZ`.
8. Set VPC Endpoint to `None`.
9. Press `Create VPC`.
10. Keep the rest of the settings as default.

![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/7af9f934-83f7-4268-b514-fd840e3311f9)

Public subnets are attached with a route table that has a destination as an internet gateway (public subnets should have a route table with internet gateway attached to it, so that traffic flows into the public subnets)

![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/5b0edd09-9322-4bc8-b920-47f78ef234fb)

Also, AWS gives you two private subnets that have two different route tables and it is attached to a VPC endpoint for the S3 bucket. This project has nothing to do with the VPC endpoint and It should remove this from the configuration.

## Create Auto Scaling Group

1. Search for EC2 in the AWS Console and navigate to the "Auto Scaling Group" option on the left-hand side.

2. Select "Create Auto Scaling Group" using a launch template named 'launch-prod' with the description 'app deploy in pvt subnet.'

3. Choose the Amazon Linux 2 AMI and a Free Tier Eligible instance type.

4. Select or create a Key Pair, and set up a new security group named 'aws-prod-sec' with inbound rules for SSH (port 22) and the application (port 8000).

5. Launch the Auto Scaling Group in the previously created VPC ('prod-environ-vpc').

6. Add Security Group Rules for SSH and the application.

7. Create the Launch Template.

8. Navigate to the EC2 dashboard, then the Auto Scaling Group section. Create a new Auto Scaling Group using the previously created Launch Template.

9. Choose 'prod-environ-vpc,' select private Availability Zones, and set desired, minimum, and maximum capacity.

10. Create the Auto Scaling Group.

## Confirm Auto Scaling Group

1. After some time, check the EC2 dashboard to confirm that the Auto Scaling Group has provisioned two running EC2 instances.

## Create a Bastion Host

1. In the EC2 dashboard, launch a new instance called 'bastion-host' using Amazon Linux 2, t2.micro instance type, and the Key Pair used for EC2 instances.

2. Place the Bastion Host in a Public Subnet, enable Auto-assign Public IP, and create a security group named 'bastion-host-sg' allowing SSH.

3. SSH into the Bastion Host and ensure the agent is running, add the Key Pair to the agent, and connect to the Bastion Host.

## SSH into a Private EC2 Instance

1. Get the private address of an instance in the application tier and SSH into it using agent forwarding.

## Create Application in Private Instance

1. Create an HTML page using `vim index.html` and launch a Python server with `python3 -m http.server 8000`.

## Create Load Balancer

1. In the EC2 dashboard, select Load Balancer and choose Application Load Balancer.

2. Name it 'aws-prod-example,' set it as internet-facing, and configure it for HTTP traffic on port 8000.

3. Ensure it's in the same VPC, set security group to 'aws-prod-sec,' and create a target group named 'aws-prod-tg.'

4. Register the two instances in the target group on port 8000.

5. Once completed, copy the DNS name and paste it into the browser to access the deployed application.

## Clean Up

Remember to terminate all instances and resources to avoid incurring charges beyond the free tier limits.
