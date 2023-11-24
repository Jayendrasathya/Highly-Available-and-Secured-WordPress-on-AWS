# Highly Available and Secured WordPress on AWS

# Concept 
Goodbye old unreliable all-in-one LAMP servers for WordPress! Our new way makes sure WordPress stays available by spreading across different Availability zones and being ready to handle more traffic if needed with Auto scaling.

Our new setup uses a strong RDS Database and a tough EFS File system that connects smoothly to WordPress in the Auto Scaling Group. We'll make things faster by using RDS Read Replica and Elasticache for quick database access and storing stuff for faster access.

For safety, we'll keep all the important stuff in Private subnets and only let trusted AWS Resources connect. A strong Application Load Balancer will send visitors to safe WordPress servers. To make things faster and more secure, we'll use Cloudfront and connect WordPress to our own web address using Route 53.

Lastly, for keeping the servers running smoothly, we'll use a Bastion Host for special access and another NAT GATEWAY to safely connect the servers to the internet for updates while keeping them protected.
# Architecture 
<img width="1078" alt="architecture" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/1b73fef1-6563-44f2-8d22-252cf6773957">

# Let's get started!
# Step 1 — Setup VPC and Subnets
<img width="1440" alt="VPC" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/9dfc7eb1-910d-463f-82b6-f4baa1b6f29e">
In this demo, I chose ap-south-1(Mumbai) region, but you can pick up any region that is near you for fast access and low latency.

Remember to deploy in at least 2 Availability Zones, with a minimum of one private and one public subnet in each. You can also limit the CIDR for this demo to reserve IPv4 addresses for other purposes.
# Step 2 — Deploy a Bastion host in a public subnet
A Bastion host will be deployed in a public subnet, and through this Bastion host, we will SSH to WordPress servers placed in private subnets.
Go to AWS EC2 and create a free tier Amazon Linux instance. For the Network setting, remember to place it in a public subnet and enable Auto-assign public IP so that we can connect to the Bastion host instance.
<img width="1440" alt="bastion" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/5dfd973c-391c-46a1-a22b-fc242fa6f67a">

# Step 3 — Deploy a temporary WordPress instance in a private subnet
We will deploy this temporary EC2 instance and install all the software with the necessary configurations to make a perfect WordPress server. Then we will create a launch template from this server for Auto Scaling.
<img width="1440" alt="temp-wp-instance" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/833171e8-ef3f-4e13-af05-2db233083a10">
Just go to AWS EC2 and create a free tier Amazon Linux instance. Remember to deploy it in a private subnet of the same Availability Zone with the Bastion host. Then we create a new Security Group for all WordPress servers that allow SSH inbound connection, with Source from the Security Group of the Bastion host.

We will install all software and add more rules to WordPress servers’ Security Group later when we add more AWS services such as RDS and EFS.

# Step 4 — Create a NAT Gateway
At the moment, the WordPress instance in the private subnet only can be SSH from the Bastion host, but the instance itself is unable to connect to the Internet to download software or make critical updates.
<img width="1440" alt="NAT-eip" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/deb8d8ea-ccfb-47d2-8b8c-c2e41aa88ae4">
To solve the issue, we can either deploy a NAT instance or a NAT Gateway. The latter is more popular because it is highly available, more robust, and managed by AWS.

In our demo, we will deploy a NAT Gateway in the same public subnet with the Bastion host and assign an Elastic IP for the NAT Gateway.
<img width="1440" alt="4  adding RT" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/8186807e-f9b9-4920-a46a-fd2008e187f7">
After deploying NAT Gateway, just remember to configure the existing route table of the private subnet that hosts the temporary WordPress server. Add a rule to allow Internet connection (Destination 0.0.0.0/0) via the newly created NAT Gateway.

To achieve high availability in your architecture, you can consider creating an additional NAT Gateway in the remaining public subnet. Configure the route table of the private subnet in the same Availability Zone to direct traffic through this new NAT Gateway. This redundancy ensures that if one NAT Gateway becomes unavailable, the other one can handle the traffic seamlessly, maintaining high availability for your system.

# Step 5 — Create an RDS Database
The database is a critical component in WordPress to store login credentials, pages, posts, etc. Relational databases such as MySQL or PostgreSQL are popular choices for WordPress.
<img width="1440" alt="5  front" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/31fc9d04-2527-4099-a820-ae43ebba4daf">

Before creating the database, we will prepare a dedicated RDS Security Group. Simply create a new Security Group, and in the inbound rules, select MYSQL/Aurora with Source from the Security Group of the WordPress app. With this option, only WordPress servers can access the RDS.
<img width="1440" alt="5 secgrp" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/b94d2ec8-8255-47e1-9218-87448cdb6a92">
Now we go to RDS and create a Database. In this demo, we will choose free-tier RDS MySQL and remember to save the database credentials, such as the Master username and Master password for later use.
<img width="1440" alt="5  name,pass" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/daf6839f-6f9b-49fa-a7b9-d6eabda66583">
To make it more secure, disable public access and deploy the RDS database in the same Availability Zone with the Bastion host and temporary WordPress instance.
<img width="1440" alt="5  VPC" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/d56312c4-82e4-4d06-a47a-7d2f8a4bc1d0">
It will take a while for the database creation. Meanwhile, we can now prepare a shared drive for the WordPress application.


