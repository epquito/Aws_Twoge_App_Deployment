# Architecture Diagram
![Alt Text](https://github.com/epquito/Assesment-1/blob/main/Assesment-1.png)

# Assessment-1
During this assessment, we will learn how to properly deploy an EC2 instance for a Flask application that will use both Gunicorn and NGINX.
## Development Tools
- AWS Account
- WSL2 or any Linux distribution
- DBeaver or any database management tool
# Creating VPC Network

1. Within the AWS Console, type "VPC" in the search bar.
2. Click on "Create VPC."
   - Choose "VPC only."
   - Optionally, add a name tag.
   - Enter an IPv4 CIDR block (e.g., 10.0.0.0/24).
   - Choose "No" for IPv6.
   - Set "Tenancy" to default.
3. Click "Create VPC."

# Creating Subnets for VPC

1. Click "Create Subnet."
2. Choose the VPC ID created earlier.
3. Enter a subnet name.
4. Select the availability zone.
5. Enter an IPv4 CIDR block.
6. Add more subnets (create two public and two private subnets).
   - Repeat steps 3-6 for each new subnet.

# Create Internet Gateway (IGW)

1. Click on "Internet Gateways."
2. Create an Internet gateway.
3. Enter a name and click "Create."
4. Attach the IGW to the VPC.
   - Choose the VPC created.
# Create Route Table
1. Click on "Route tables."
2. Create a route table.
3. Enter a name.
4. Select the VPC created.
5. Click "Create."
6. Click on "Subnet associations."
   - Select subnets to associate with the route table.
7. Click on "Routes."
8. Edit routes and add a route to the IGW for public subnets.
   - Repeat for private subnets but use a NAT gateway instead.
# Create NAT Gateway for Private Route
1. Click on "NAT Gateways."
2. Create a NAT gateway.
3. Enter a name.
4. Select a public subnet.
5. Set connectivity to "Public."
6. Allocate an Elastic IP.
7. Click "Create NAT Gateway."
# Associate NAT Gateway to Private Route Table
1. Select the public route table.
2. Edit the inbound rule to allow traffic from the private route table.
   - Allow traffic originating from the private subnet CIDR block.
3. Go to the private route table.
4. Establish a route for outbound traffic to the internet through the NAT Gateway.
   - Add a route with destination `0.0.0.0/0` (or the specific CIDR block you want) pointing to the NAT Gateway.
5. Save the changes.
# Deploying EC2
1. Navigate to the search bar, type "EC2," and select "Launch Instance."
2. Enter a name for your instance.
3. Under "Application and OS Images," choose "Amazon Linux" as the OS.
   - Amazon Machine Image (AMI): Choose "Amazon Linux 2 AMI."
4. Select the instance type (e.g., t2.micro).
5. Configure the Key Pair by selecting a key pair associated with your account.
6. In "Network Settings," click "Edit":
   - VPC: Choose either the default VPC or another created VPC.
   - Subnet: Choose a public subnet.
   - Auto-assign IP: Disable.
7. Firewall/Security Group:
   - Create a new Security Group.
   - Add a description and ensure the security group includes SSH/HTTP/HTTPS.
   - Source Type: Custom.
   - Source: 0.0.0.0/0.
8. Click "Launch Instance."
# Creating IAM Role for EC2 to Access Your S3 Buckets
1. In the search bar, type "IAM," select "Roles" on the left side, and click "Create role."
2. Choose the trusted entity type: AWS service, use case: EC2, and click "Next."
3. Under "Permissions Policies," use the search bar, type "S3," select "AmazonS3FullAccess," and click "Next."
4. In "Role Details," enter a role name and click "Create role."
5. In the search bar, type "EC2," choose your EC2 instance, click "Actions," navigate to "Security," and click "Modify IAM role."
6. Select the IAM role created in step 4.
# Deploying RDS
1. In the search bar, type "RDS" and select "Create database."
2. Choose "Standard create," Engine type: PostgreSQL, Engine Version: 15.3-r2, and Templates: Free tier.
3. Under "Settings":
   - DB instance identifier: Create and enter the name of the database.
   - Master username: Change the username (recommended).
   - Master Password: Enter a personal password and confirm it.

4. In "Instance configuration," choose db.t3.micro.
5. Under "Storage," leave default values, and deselect "Enable Storage Autoscaling."
6. In "Connectivity":
   - Don't connect to an EC2 compute resource.
   - IPv4: Choose the same VPC you selected for the EC2.
   - DB subnet group: Leave default.
   - Public Access: Yes.
   - VPC security group: Create a new one and enter a name.
   - Additional configurations:
      - Database port: 5432.

7. Database authentication: Select "Password authentication."
8. Leave all values under "Monitoring" as default.
9. In "Additional configuration":
   - Database option: Initial database name (Enter the database name associated with your Flask app).
   - Select "Create Database."
   - Initialization may take 5-10 minutes.

10. Once the database is created, select it and go to the "Connectivity & security" tab.
11. Select "VPC Security groups," choose the security group you created for your database, and click "Edit inbound rules."
12. Ensure the rule looks like this:
    - Security group rule Id.
    - Type: PostgreSQL.
    - Protocol: TCP.
    - Port Range: 5432.
    - Source: Custom.
    - Destination: Security group of your EC2 instance previously created.
# Creating an S3 Bucket to Hold Static Files
1. In the search bar, type "S3" and select "Create bucket."
2. Enter a unique bucket name and choose the AWS Region assigned to you.
3. Under "Object Ownership":
   - Disable ACLs.

4. In "Block Public Access settings for this bucket":
   - Unclick "Block all public access."
   - Acknowledge the action.

5. Enable "Bucket versioning."
6. Click "Create bucket" and leave everything else as default.

7. Click on your bucket again.
8. On the "Object" page, upload the static files you want to upload.

9. On the tab, click "Permissions."
10. Create a bucket policy that allows EC2 to have access to objects/object versions. Use the following JSON snippet:

```json
{    
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2Instance",
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
      ],
      "Resource": "arn:aws:s3:::s3-bucket-name/*",
      "Condition": {
        "StringEquals": {
          "aws:userid": "arn:aws:iam::role-ID/Name of role"
        }
      }
    }
  ]
}
```

# SSH into EC2 to Deploy Flask Application with Gunicorn/Nginx

1. Inside the AWS console, go to the search bar and type "EC2."
2. Select the EC2 instance you want to SSH into.
3. Once selected, click on "Connect" and then click on "SSH client."
4. At the bottom, under "Example," copy the SSH command.
5. In your terminal, navigate to the directory containing your key pair (it should end with .pem).
6. Paste the copied SSH command. If it asks for confirmation with "yes" or "no," type "yes" and press Enter.
7. If the command for SSH ends with "root@ec2-ip," change "root" to "ec2-user."

# Installing Dependencies and Deploying Flask Application

8. Update the system and install Git:
```bash
sudo yum update -y
sudo yum install git -y
```
9. Install Nginx:
```bash
sudo amazon-linux-extras install nginx1
```
10. Clone the GitHub repository:
```bash
git clone https://github.com/chandradeoarya/twoge.git
cd twoge
```
11. Install Python dependencies:
```bash

sudo yum install python3-pip -y
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
12. Edit the Flask app configuration:
```bash
sudo vim app.py
#Update line 11 with the appropriate RDS database URI.
```
13. Create and configure Gunicorn service:
```bash
cd /etc/systemd/system
sudo vim twoge.service
#Copy and paste the provided service configuration.
```

```bash      
  [Unit] 
  Description=Gunicorn instance to serve twoge
  Wants=network.target
  After=syslog.target network-online.target
  [Service]
  Type=simple
  WorkingDirectory=/home/ec2-user/twoge
  Environment="PATH=/home/ec2-user/twoge/venv/bin"
  ExecStart=/home/ec2-user/twoge/venv/bin/gunicorn 	app:app -c /home/ec2-user/twoge/gunicorn_config.py
  Restart=always
  RestartSec=10
  [Install]
  WantedBy=multi-user.target
  ```
14. Enable and start the Gunicorn service:
```bash
sudo systemctl enable twoge
sudo systemctl start twoge
sudo systemctl status twoge
```
15. Configure Nginx:
```bash
cd /etc/nginx/sites-available
sudo vim twoge_nginx
#Copy and paste the provided Nginx configuration.
```
```bash     	
  server {
    listen 80;
    0.0.0.0/0;
    location / {
       proxy_pass http://0.0.0.0:9876;  # Assuming Gunicorn is running on port 8000
       include /etc/nginx/proxy_params;
            }
}
```
16. Create a symbolic link and test Nginx configuration:
```bash
sudo ln -s /etc/nginx/sites-available/twoge_nginx /etc/nginx/sites-enabled/
sudo nginx -t
```
17. Reload Nginx and start it:
```bash
sudo systemctl reload nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

# Create an Image of the Instance
1. Select your EC2 instance.
2. Click on "Actions" -> "Image and templates" -> "Create Image."
3. Enter a name for your image.
4. Add a description (optional).
5. Leave all other settings as default.
6. Click on "Create Image."
# Create Launch Template using the Image created
1. Navigate to "Launch Templates" and click on "Create launch template."
2. Enter a name and template version.
3. Under "Application and OS images":
   - Choose "My AMIs."
   - Select "Owned by me."
   - Choose the Amazon Machine Image (AMI) created from the EC2 instance.
4. Under "Instance Type," select "t2.micro" and choose the key pair used when creating the EC2 instance.
5. For "Network Settings," leave the subnet as default, and for "Firewall/Security group," select the existing security group created for the EC2 instance.
6. Leave everything else as default, except for "Advanced details":
   - Under "IAM instance profile," select the IAM role that allows EC2 to have access to the S3 bucket.
7. Click on "Create launch template."
# Creating Target Group
1. Navigate to "Target groups" and click on "Create target group."
2. Under "Instances":
   - Enter a target group name.
   - Set "Protocol" to the desired protocol (e.g., HTTP or HTTPS).
   - Set "Port" to the target port (e.g., 9876).
   - Choose "IPv4."
   - Select the VPC used for the EC2 instance.
   - Leave all other settings as default.
   - Click "Next."
3. Under "Register targets":
   - Select the EC2 instances you want to register.
   - Include as pending below.
   - Click "Create target group."
# Create Load Balancers
1. Navigate to "Load Balancers" and click on "Create load balancer."
2. Choose "Application Load Balancer."
3. Enter a Load Balancer name.
4. Select the VPC (the same one used for the EC2 instance).
5. Under "Mappings," select Availability Zones (AZ).
6. Create a Security Group (SG) that includes HTTP/HTTPS and a custom TCP with a port range of 9876.
7. Proceed to "Listeners and routing."
8. Select the Target Group previously created.
9. Click "Create Load Balancer."
# Create Auto Scaling Group (ASG)
1. Navigate to "Auto Scaling groups" and click on "Create Auto Scaling group."
2. Enter a name for the Auto Scaling group (ASG).
3. Select the Launch template previously created.
4. Choose the VPC (the same one used for the EC2 instance).
5. Select the Availability Zones.
6. Attach to an existing load balancer:
   - Choose the Load Balancer.
7. Under "Health checks," turn on ELB health checks:
   - Set Health check grace period to 120.
8. Click "Next."
9. Under "Group Size":
   - Recommended settings: Desired (2), Minimum (1), Maximum (3).
10. Leave Scaling Policies as "none."
11. Add Notification:
    - Create a topic.
    - Enter a topic name.
    - Add your email as recipients.
    - Choose event types (Launch, Terminate).
    - Confirm subscriptions.
12. Click "Next" until you reach "Create Auto Scaling group."
13. Click "Create Auto Scaling group."
# Create Auto Scaling Group (ASG) Dynamic Scaling Policy
1. Select the Auto Scaling Group (ASG).
2. Go to "Automatic Scaling" -> "Dynamic scaling policies" -> "Create Dynamic scaling policy."
3. Choose the policy type as "Simple Scaling."
4. Enter a policy name.
5. For CloudWatch alarm, click "Create CloudWatch."
6. Select the metric:
   - Metric -> Select Metric -> EC2 -> By ASG -> Choose AutoScalingGroupName to populate the ASG you just created.
   - Select your specific ASG with the metric value of "CPUUtilization."
   - Click "Select Metric."
7. Specify Metric and Conditions:
   - For Static (average), set Period to 1 minute.
   - Condition: Threshold (Static).
   - Whenever CPUUtilization is (Greater/Equal) than (Enter Threshold).
8. Notifications:
   - Create a topic.
   - Enter a topic name.
   - Enter an email endpoint.
   - Create the topic.
9. Name and Description:
   - Enter an alarm name.
   - Click "Next."
   - Click "Create Alarm."
10. Go back to "Create Dynamic Scaling Policy."
11. Click the Refresh button next to CloudWatch Alarm and select the newly created alarm.
12. Take Action: Add / 1 / capacity units.
13. Wait (60 seconds).
14. Click "Create."
# Great job! It looks like you've successfully deployed a Flask application with Gunicorn and Nginx on an EC2 instance with ELB and ASG, enabling high scalability and resiliency. If you have any further questions or need additional assistance, feel free to ask!
        

        


