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



