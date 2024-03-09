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
8. Set VPC Endpoint to `None` (This is the spot where we remove the default created config of the VPC endpoint for the S3 bucket).
9. Press `Create VPC`.
10. Keep the rest of the settings as default.

### Explanation of previews

![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/7af9f934-83f7-4268-b514-fd840e3311f9)

AWS has created public-private Subnets in the `US East 1A AZ` and `US East 1B AZ`. Public subnets are attached with a route table that has a destination as an internet gateway (public subnets should have a route table with internet gateway attached to it, so that traffic flows into the public subnets)

![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/5b0edd09-9322-4bc8-b920-47f78ef234fb)

Also, AWS gives you two private subnets that have two different route tables and it is attached to a VPC endpoint for the S3 bucket. This project has nothing to do with the VPC endpoint and It should remove this from the configuration (Done in step 8 above).


## Step 2: Create Auto Scaling Group

1. Search for EC2 in the AWS Console and select `Auto Scaling Group` in the left pane.
2. Create a new Auto Scaling Group using a launch template. Before creating the auto-scaling group, you have to have a lunch template. So press: `Create a lunch template`.
3. Name the Launch Template `aws-prod-exaple` with a description of `app deploy in pvt subnet`.
4. For the AMI Select the `Ubuntu (Ubuntu Server 22.04 LTS (HVM), SSD Volume Type)`.
5. Select a `Free Tier Eligible` instance type.

   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/efc302b1-18ee-4713-9c83-17720401bc7e)

7. Choose or create a Key Pair.
8. In network settings for the Firewall (security groups) press the `Create security group` and set up a new security group named `aws-prod-sg`.
9. Select the correct VPC above that was created instead of the default VPC: `aws-prod-example-vpc`
10. Configure inbound rules for SSH (port 22) and the application (port 8000).
11. Create the Launch Template.
12. Navigate to the EC2 dashboard, go to the Auto Scaling Group section, and create a new Auto Scaling Group using the previously created launch template.
13. Choose the 'prod-environ-vpc' VPC.
14. Select the private Availability Zones for deployment.
15. Skip Load Balancing configuration.
16. Set the desired capacity to 2 instances, minimum capacity to 1, and maximum capacity to 4.
17. Create the Auto Scaling Group.

**Note:** Auto Scaling Group creation may take some time.

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
