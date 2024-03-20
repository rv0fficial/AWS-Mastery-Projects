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
5. Name the VPC as `aws-prod-example-vpc` (Name tag auto-generation).
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
3. Name the Launch Template `aws-prod-template` with a description of `app deploy in pvt subnet`.
4. For the AMI Select the `Ubuntu (Ubuntu Server 22.04 LTS (HVM), SSD Volume Type)`.
5. Select a `Free Tier Eligible` instance type.
   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/efc302b1-18ee-4713-9c83-17720401bc7e)
7. Choose or create a Key Pair.
8. In network settings for the Firewall (security groups) press the `Create security group` and set up a new security group named `aws-prod-sg`.
9. Select the correct VPC above that was created instead of the default VPC: `aws-prod-example-vpc`
10. Select `Add security group rule` & configure inbound rules for SSH (`port 22`) and the application (`port 8000`).
11. No other modification, just press: `Create launch template`.
12. Navigate to the `EC2 dashboard`, go to the `Auto Scaling Group` section.
13. Select `Create Auto Scaling Group` and set the Launch Template to be the one that was just created: `aws-prod-template`.
14. Scroll down and then Click `Next`.
15. Choose the `aws-prod-example-vpc` VPC.
16. Select the created `two private AZs` for deployment.
17. Scroll down and then Click `Next`.
18. Skip the `Load Balancing` configuration, no other changes need to be made, scroll down and then Click `Next`.
19. Set the `desired capacity` to `2` instances, `minimum capacity` to `1`, and `maximum capacity` to `4`.
    You can configure the scaling option which means when to scale up or down, but it's not focused at this point.
    Keep the option starting with `No` and Click `Next`.
22. Click `Next` without adding anything to the `Add notification` section.
23. Click `Next` without adding anything to the `Add tags` section.
24. Scroll down and then Click `Create Auto Scaling group`.

**Note:** Auto Scaling Group creation may take some time.

## Confirm Auto Scaling Group

1. After some time, check the EC2 dashboard to confirm that the Auto Scaling Group has provisioned two running EC2 instances.

   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/0b4bc1cc-cee2-44ab-b74f-5be207ef7a2d)

   Check the instances are in the correct AZ which is one should be in `us east 1a` and the other one in `us east 1b`.

## Create a Bastion Host

Before creating the Application Load Balancer, the application needs to be installed on the servers. To do that we will ssh into the instances and install it. When you try to log into these instances, you will notice that this instance does not have a public IP address. I have not provided any public IP addresses as these instances should be secure.

That's where Bastion host comes into the picture. It acts as a mediator between your private subnet and the external persons or the public subnet. I'll create a Bastion host and access the private subnet.

1. In the EC2 dashboard, launch a new instance called `bastion-host` using `Ubuntu`, `t2.micro` instance type, and the Key Pair used for EC2 instances that were used earlier.
2. For the VPC ensure that the one selected is the one where the Ec2 Instances are located (`aws-prod-example-vpc`), else the Bastion Host will not be able to do its job.
3. `Enable` Auto-assign Public IP as without this it will be of no use.
4. Place the Bastion Host in one of the Public Subnet: `public1-us-east-1a` or `public2-us-east-1b`.
5. Create a security group named `bastion-host-sg` allowing SSH.
7. Scroll down and then Click `Launch instence`.

**Note:** Make sure this EC2 instance is inside the same VPC that was created in this project. And enable the auto-assign public IP as well.

## SSH into a Private EC2 Instance

To SSH into the private instances, we first need to connect to our Bastion host instance. 
From there, we'll be able to SSH into the private instance. Make sure that the PEM file is present on the Bastion host. 
Without it, you won't be able to SSH into the private instance from the Bastion host.

1. Open a terminal window on your local machine.
2. Copy the PEM file (or the private key of the VM that was created by the auto-calling group) to the Bastion host using the SCP command.

   Dummy: `scp -i <Bastionhost'sKeyPair.pem> <Instence'sKeyPair.pem> ubuntu@<bastion-ec2-instance-ip>:/home/ubuntu`
   - `Bastionhost'sKeyPair.pem`: Specifies the path to Bastion host's private key file for authentication.
   - `Instence'sKeyPair.pem`: Specifies the path to private subnet Instence's private key file that needs to be sent to the Bastion host (that you want to copy).
   - `:/home/ubuntu`: The key which is above to send (or copy) to the bastion host, will reside in this location.

   Used: `scp -i Downloads/tween-prod-nvir.pem Downloads/tween-prod-nvir.pem ubuntu@18.234.83.17:/home/ubuntu`
   
   The above command will copy the PEM file from your computer to the Bastion host. Once the file is successfully copied, move on to the next step.
4. Permission issue came into play while trying to ssh the created instances (ssh from bastion to private subnet’s ec2).

   Dummy: `chmod 600 <Instence'sKeyPair.pem>`
   - `Instence'sKeyPair.pem`: The key sent to `/home/ubuntu` on the Bastion host.
   
   Used: `chmod 600 tween-prod-nvir.pem`
5. SSH into the private subnet’s ec2 from Bastion host using the following command.
   ```
   ssh -i tween-prod-nvir.pem ubuntu@10.0.157.146
   ```
   10.0.157.146 is a private IP of created ec2 instances in the private subnet.

## Create Application in Private Instance

now I can log into the private instance as well and all that I'll do is install a very simple Python application.

1. Create an HTML page using `vim index.html`.
   
   within this file, we are going to create a very basic HTML page. You can put whatever the fancy HTML pages as well. Finally, save the file and exit.
2. Launch Python HTTP server on port 8000 to deploy your application on the private instance with `python3 -m http.server 8000`.
   
   Now, your application is deployed on the private instance on port 8000.

**Note:** We intentionally deployed the application on only one instance to check if the Load Balancer will distribute 50% of the traffic to one instance (which will receive a response) and 50% to another instance (which will not receive a response).

## Create Load Balancer

1. In the EC2 dashboard, select `Load Balancer` and choose `Application Load Balancer`.
2. Name it `aws-prod-alb`, set Scheme as `internet-facing`
3. Keep the IP address type as it is: `IPv4`
4. Ensure that it is placed in the same VPC where the instances have been deployed: `aws-prod-example-vpc`.
5. Set the Mapping for both `Public AZ` only: `aws-prod-example-subnet-public1-us-east-1a` and `aws-prod-example-subnet-public2-us-east-1b`.
6. Select a Security Group that allow SSH, HTTP traffic and port 8000 traffic: `aws-prod-sg`.
   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/2896df2d-0646-484d-b07a-c517b1a5064a)
7. Create a Target group where you will Define which instances should be accessible: `Create target group`.
   - Choose a target type: `Instances`
   - Target group name: `aws-prod-tg`
   - Protocol | Port: `HTTP`|`80`
   - IP address type: `IPv4`
   - Then keep the rest as default, Scroll down and then Click `Next`
9. Select the instances that you are trying to access using port 8000 and Click `Include as pending below`.
   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/38269844-14f1-4ec2-a078-69ee743d725f)
10. Scroll down and then Click `Create target group`. Then let's add this target group to the load balancer
11. Select the `Default action` as created target group: `aws-prod-tg`
   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/18655df3-1e55-4c22-9918-03121d09c717)
12. Then keep the rest as default, Scroll down and then Click `Create load balancer`.
13. Copy the DNS name and paste it into the browser and be able to access the website. 
   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/3b227e50-bd7f-476d-945d-905d4cf190e2)
   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/ea666c46-f0d5-4892-bd17-a4e2b63bddaf)
   ![image](https://github.com/rv0fficial/AWS-Mastery-Projects/assets/147927710/baae794d-0a41-450d-bfda-9ff235bbed48)

Now we have deployed the Application securely in a Private instance. However I have intentionally not deployed in one of the instances, so sometimes you will get an error.

## Clean Up

When you are finished with this configuration, you can delete it. Before you can delete the VPC, you must delete the Auto Scaling group, terminate your instances, delete the NAT gateways, and delete the load balancer. For more information, see [Delete your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/delete-vpc.html).
