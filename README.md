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


# Step 6 — Create an EFS shared drive
As we leverage Auto Scaling to adjust the number of WordPress instances based on demand dynamically, it is advantageous to utilize an EFS shared folder to store all configurations and data for the WordPress application. This ensures that any new server joining the Auto Scaling Group can immediately access the EFS shared folder and the RDS database to handle incoming traffic efficiently.
<img width="1440" alt="6  sg" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/0706ede7-71e3-44d5-aa8e-f725acd835ba">
First, we need to create a new Security Group for EFS and allow incoming NFS traffic directly from the Security Group of WordPress servers.
<img width="1440" alt="6 gen" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/3ce4200d-a59d-4e1d-afd0-c81fe1a2672c">
After that, we just create our own EFS. Remember to deploy on private subnets and attach to the newly created EFS’s Security Group.
<img width="1440" alt="6 network" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/ea17a092-2e84-4f70-8695-4e1ee8ef49b6">
Creating the EFS may require some time, but once it’s completed, you’ll receive the essential information to attach the EFS drive to our WordPress instance. Make sure to take note of the highlighted mounting DNS information, as you will need it for future steps.
<img width="1440" alt="6 mount tar" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/88c3cc66-a4ce-41ca-9b6e-0d29b137954c">

# Step 7 — Prepare the WordPress instance
With the completion of the essential infrastructure setup, including the Bastion host, temporary WordPress server, RDS database, and EFS shared drive, we are now ready to create an impeccable WordPress server!

To establish an SSH connection with the WordPress server in the private subnet, you will require two things:

1.The private IP of the WordPress server

2.The previously saved WordPress key pair on your computer.

<img width="1440" alt="7  private instance" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/5e82d524-15b6-4a86-9a0c-82e47d878674">


To access the key pair, simply open it with any text editor. In my case, I use TextEdit on my Mac to view the content. Copy the entire key from the file.


<img width="1440" alt="7  copied key" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/58831a5a-0c17-440c-bd2a-992b948cfc65">


Now, connect to the Bastion host instance via EC2 Instance Connect in the browser.


<img width="1440" alt="7  ssh client" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/a6a14a26-c4c7-4a29-aeaf-0f66724d3e59">

Insert the following command to create a new key pair in the Bastion host terminal. Paste the copied key pair and then save it.

#nano app-key.pem

After recreating the key pair, we will update its permissions and then SSH to the temporary WordPress instance using its private IP. Make sure to replace “10.0.10.13” with the private IP of your WordPress instance.

#sudo chmod 400 app-key.pem

#ssh -i "wp-project.pem" ec2-user@10.0.10.13

<img width="1440" alt="7  connected pic" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/3b7faf33-670c-45de-a526-cfc16930623a">

Next, we’ll proceed with updating the instance and installing essential software, including Apache, PHP, and MySQL.

#sudo yum update -y

#sudo yum install php -y

#sudo yum install php-mysqli -y

#sudo yum install mariadb105 -y

Once completed, we’ll activate these services and configure them to start automatically after a system restart.

#sudo systemctl start httpd

#sudo systemctl enable httpd


#sudo systemctl start php-fpm

#sudo systemctl enable php-fpm

Then we will create some permits to access the Apache public folder.

#sudo usermod -a -G apache ec2-user

#sudo chown -R ec2-user:apache /var/www/html

#sudo chown -R apache:apache /var/www/html

Using the recently created EFS shared drive, we will proceed to mount it to the server’s root folder for the website. Make sure to replace “fs-0b3889c8aecf78a5f.efs.ap-south-1.amazonaws.com:/” with the DNS of your EFS folder. If you’re unsure where to locate it, refer to the instructions at the end of Step 6.

#sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-0b3889c8aecf78a5f.efs.ap-south-1.amazonaws.com:/ /var/www/html/

To ensure that the EFS shared folder is automatically mounted to the WordPress server after a restart, enter the following command.

#sudo nano /etc/fstab

<img width="1440" alt="7  fstab entry" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/9adf7083-7251-4e24-99f9-6f1194dfa919">

Append the following line to the end of the file and save it. Make sure to replace “fs-0b3889c8aecf78a5f.efs.ap-south-1.amazonaws.com:/” with the DNS of your EFS folder. If you’re unsure where to find it, refer to the end of Step 6.

#fs-0b3889c8aecf78a5f.efs.ap-south-1.amazonaws.com:/ /var/www/html/ nfs defaults,_netdev 0 0

To verify that the EFS drive has been successfully mounted, enter the following command in the terminal. You should observe the EFS drive being mounted to “/var/www/html,” which represents the root folder of the website.

#df -T -h

<img width="1440" alt="7  df" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/592f3a0a-da8c-4c87-88c5-ecee5c858ea7">

Next, we will download WordPress, unzip the downloaded file, and install it on the temporary server using the following commands.

#wget https://wordpress.org/latest.zip

#unzip latest.zip

#sudo cp -R wordpress/* /var/www/html/

Now, the preparation for the WordPress server is complete. However, before exiting the Bastion host, we need to access RDS and create an initial database.


<img width="1440" alt="7  db endpoint" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/413f9414-739f-4573-aa0e-530c807fe359">

Insert the following command in the Bastion host terminal and replace “wp-db.ccngqvqhtcva.ap-south-1.rds.amazonaws.com” with your RDS endpoint and “admin” with your RDS Master username if you have chosen a different one

#mysql -h wp-db.cb7yjv0w5dmw.us-west-1.rds.amazonaws.com -P 3306 -u admin -p

After typing in the RDS Master password, please enter the following SQL commands. This command will create a database named “project” as the first WordPress database. Feel free to replace “project” with your preferred name if you’d like.

#create databases project;

#show databases;

<img width="1440" alt="7 maria intr" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/d9c5a913-8a10-4ac8-b84b-7f1ac91c1b3c">


# Step 8: Create a launch template for Auto Scaling

Now that we have set up a flawless WordPress instance, let’s proceed to create a launch template for future Auto Scaling.

<img width="1440" alt="8  create image" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/643c3f9c-204d-4dbe-b436-81954fe0f84a">

Simply select the temporary WordPress instance, in “Actions,” choose “Image and Templates,” and then select “Create image.”


<img width="1440" alt="8 image-int" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/c7b1dbf3-2ea2-4f30-8e31-c86710322050">

It will take a while to create a golden image of the temporary WordPress instance. Once it is done, go to EC2 and create a launch template.


<img width="1440" alt="8  launch temp" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/5d58900a-288e-4879-989f-daf307ab2a02">


<img width="1440" alt="8 myAMI" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/0a429e21-d843-41ed-b667-46aa29eca7de">

In the “Application and OS Images” option, select “My AMIs” and then choose your newly created WordPress instance image.


<img width="1440" alt="8  instance type" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/56c781ce-2bc5-453b-b9a2-efa90cba74fd">

For the instance type, choose t2.micro as it is eligible for Free tier. Select the existing WordPress key pair for the launch template and the existing Security Group for the WordPress instance. 


# Step 9 — Deploy an Auto Scaling group

Now that we have our WordPress launch template, launching WordPress instances is a breeze. Simply navigate to the EC2 console and select “Auto Scaling Groups.” From there, choose the recently created launch template. This allows you to effortlessly launch multiple WordPress instances with ease.


<img width="1440" alt="9  choose LT" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/1cb0db26-a70f-4ad5-bf34-a7d3a758100f">

In the “Network” option, select the private subnets in both Availability Zones. This ensures that the new WordPress instances will be launched in those specific subnets, providing the desired network configuration.

<img width="1440" alt="9 lauch opt" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/406cf330-30ff-413f-9483-c0753da50a41">

In the “Group size” configuration, set the desired capacity to 2. This will ensure that 2 instances are launched, with one instance in each private subnet.

As for scaling policies, you have the flexibility to choose based on your specific requirements. If you prefer, you can leave it as “None” so that the default scaling policy will rely on the instance health check.


<img width="1440" alt="9  groupsize" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/4cad6760-6981-4ad7-99db-ee23111cc5df">

Please note that it may take some time for the two new WordPress servers to be provisioned in the private subnets.


<img width="1440" alt="9 autoscaled" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/7b2f6cfd-8a33-4a2e-8ad2-5e84ce0a2ec9">

Once the process is complete, you will see a total of four instances in the EC2 dashboard. These include the two instances in the Auto Scaling group, the Bastion host, and the temporary WordPress instance that was created earlier.

You can now terminate the temporary WordPress instance by accessing its terminal. You can also create an Auto Scaling group for Bastion hosts to make them highly available as well.


# Step 10 — Create an Application Load Balancer and connect with the Auto Scaling group

Now that we have completed the back-end setup of our WordPress application, the next step is to connect it with a Load Balancer.

To do this, we will create a Target Group for the Load Balancer. Choose “Instances” as the Target type and “HTTP” as the listening protocol. If desired, we can also add the “HTTPS” protocol later.


<img width="1440" alt="10  basic conf tg" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/372d43d3-960d-46bd-8ae3-8919999dec43">

Please note that when creating the Target Group for the Load Balancer, make sure not to select any running instances as targets. This is because these instances may be terminated in the future as we scale up or down our Auto Scaling group. Instead, we will have the option to attach the Auto Scaling group to the Load Balancer’s target, which will be shown later in the process.


<img width="1440" alt="10 register tar" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/42af81fc-6b3e-4e67-b2b9-3b0fbb2b9e5e">


Now, create a new Security Group for Load Balancer, allowing inbound rules for HTTP and HTTPS traffic.


<img width="1440" alt="10  sg elb" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/0b18eecf-a56b-4e98-806a-f4cd59505f7b">

Next, let’s create an Application Load Balancer. Choose the Internet-facing scheme and ensure that the Load Balancer is placed in all public subnets.


<img width="1440" alt="10  basic conf" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/b6096aad-9687-4b72-856a-9c17bf1f7eed">


<img width="1440" alt="10 network" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/92153ea6-9109-4e17-b88f-70e92d711f80">

Select the recently created Security Group for the Load Balancer and configure it to listen to both HTTP and HTTPS connections.


<img width="1440" alt="10  sg, lis" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/52229160-58b5-4d5c-978a-fa9eb48e8ebd">

After finishing the setup for the Application Load Balancer, go back to the existing Auto Scaling group and attach it to the newly created Load Balancer.


<img width="1440" alt="10  details asg" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/a9286e98-968c-415c-9b82-a87dcd124f20">



<img width="1440" alt="10  edit wp-asg" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/900e6efd-3fc4-4726-a0fd-f2f14633a06d">


To allow incoming traffic from the Load Balancer, navigate to the Security Group associated with the Auto Scaling group. Add inbound rules for both HTTP and HTTPS protocols, specifying the source as the Security Group of the Load Balancer.



<img width="1440" alt="10  last" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/d92ba63f-1388-4c9f-adfa-55a49f21d98d">




To test the new Load Balancer, copy its endpoint and paste it into your browser. You should see a success screen confirming that the connection was successful.



<img width="1440" alt="10  test" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/9414baca-8c9a-4138-8adf-5da9c1c1c509">


# Step 11 — Create a CloudFront distribution

To set up CloudFront, navigate to the service and choose the newly created Application Load Balancer as the Origin Domain. Select HTTP as the default option for the listener protocol.



<img width="1440" alt="11  -1" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/461d9b6a-e078-46bb-a591-23d63582aed1">


For the Viewer setting, select HTTP and HTTPS, or you can choose to redirect HTTP to HTTPS. However, if you opt for the redirect option, you will need to request an SSL certificate.


Make sure to choose “GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE” as these methods allow the WordPress admin user to write, post, and configure the website.


<img width="1440" alt="11  -2" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/7f68a0a6-3b61-42f9-833f-19cca7148b33">


After the setup is complete, you will receive a CloudFront distribution domain name that you can use to access your WordPress website.


If you want to connect your own domain, which is managed by Route 53, with the CloudFront distribution, you need to add that domain name in the “Alternative domain names” field.


But I don't have any own domain, So I'm skipping the part(Route53)


<img width="1440" alt="11  -3" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/8202b964-3dcd-400a-b12d-b094222f7f0e">


I don't have any domain, So I've hosted it with loadbalancer DNS. 


# Output


<img width="1440" alt="Screenshot 2023-11-24 at 7 05 25 PM" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/91ef9b14-1d7a-457b-8a14-647c2473b16b">



<img width="1440" alt="Screenshot 2023-11-24 at 7 05 39 PM" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/ff529dc0-e13a-496c-b636-55fa1d48de06">



<img width="1440" alt="Screenshot 2023-11-24 at 7 06 00 PM" src="https://github.com/Jayendrasathya/Highly-Available-and-Secured-WordPress-on-AWS/assets/116148912/cb34a5f9-2054-433f-a689-220b4cf7fc99">











