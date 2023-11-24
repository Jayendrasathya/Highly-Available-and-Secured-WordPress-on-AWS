# Highly Available and Secured WordPress on AWS

# Concept 
Goodbye old unreliable all-in-one LAMP servers for WordPress! Our new way makes sure WordPress stays available by spreading across different Availability zones and being ready to handle more traffic if needed with Auto scaling.

Our new setup uses a strong RDS Database and a tough EFS File system that connects smoothly to WordPress in the Auto Scaling Group. We'll make things faster by using RDS Read Replica and Elasticache for quick database access and storing stuff for faster access.

For safety, we'll keep all the important stuff in Private subnets and only let trusted AWS Resources connect. A strong Application Load Balancer will send visitors to safe WordPress servers. To make things faster and more secure, we'll use Cloudfront and connect WordPress to our own web address using Route 53.

Lastly, for keeping the servers running smoothly, we'll use a Bastion Host for special access and another NAT GATEWAY to safely connect the servers to the internet for updates while keeping them protected.
# Architecture 
<img width="1078" alt="architecture" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/1b73fef1-6563-44f2-8d22-252cf6773957">



