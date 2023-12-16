# Assessment-1
During this assessment, we will learn how to properly deploy an EC2 instance for a Flask application that will use both Gunicorn and NGINX.
## Development Tools
- AWS Account
- WSL2 or any Linux distribution
- DBeaver or any database management tool
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
# SSH into Ec2 to deploy Flask application With Gunicorn/Nginx

- Inside Aws console go to the search bar and type ec2

     
- Select a ec2 you want to ssh into
     
- once selected click on connect and click on ssh client
    
- There should be all the way on the bottom "Example:" that would have a ssh comand to copy
    
- go to your terminal
      
- make sure you change directory that holds your key pair you selected for that instance it should end with a .pem

- paste the command and it will ask a yes or no to continue type yes and press enter
     
- If the command for ssh  ends with a "root@ec2-ip" change the "root" to "ec2-user"


# Once you have all the dependicies installed now you change the repository you just installed for your personal deployment
    
- sudo yum update -y
        
        
- sudo yum install git -y
        
        
- sudo amazon-linux-extras install nginx1
        
        
- git clone https://github.com/chandradeoarya/twoge.git


- cd twoge


- sudo yum intall python3-pip -y


- python3 -m venv venv


- sudo vim app.py
    line 11 : app.config['SQLALCHEMY_DATABASE_URI'] = os.environ.get('SQLALCHEMY_DATABASE_URI','postgresql://username:password@EndPoint-RDS:5432/dbName')


- source venv/bin/activate


- pip install -r requirements.txt


- cd /etc/systemd/system
               
  sudo vim twoge.service:
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

         
- sudo systemctl enable twoge


- sudo systemctl start twoge


- sudo systemctl ststus twoge


- cd /etc/nginx


- sudo mkdir sites-available


- sudo mkdir sites-enabled


- cd sites-available

- sudo vim twoge_nginx:
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
        
- sudo ln -s /etc/nginx/sites-available/twoge_nginx /etc/nginx/sites-enabled/

- sudo nginx -t


- sudo systemctl reload nginx


- sudo systemctl enable nginx


- sudo systemctl start nginx

# Create A Image of the instance 

- Select instance -> Actions -> Image and templates -> Create Image -> Enter image name->
            Add discription -> Leave everything else default -> Create Image


# Create Launch Template using the Image created 

- Launch Templates -> Create launch template -> Enter name -> Template version ->
        Application and OS images -> My AMIS -> Owned by me -> Amazon Machine Image (Select the Image we created of the instance)

- Instance Type(t2.micro) -> Key pair(select the one used when ceating Ec2) 

- Network Setting -> Subnet leave default -> Firewall/Security group(Select existing security group created for ec2 instance)    

- Leave everything default Except for Advanced details -> IAM instance profile -> Select The Iam role we created that allows ec2 to have access to the s3 bbucket -> launch template
            
# Creating target group

- Target groups -> Create target groups -> Instances -> Enter target group name -> Protocol port(9876) -> IPv4 -> Select Vpc(Vpc we used for the Ec2 instance) ->                 Leave everything else default -> next

- Register targets -> Select Ec2 instances you want to register -> Include as pending below -> Create target group

# Create Load Balancers  

- Load Balancers -> Create load balancer -> Application Load Balancer -> Load Balancer  name -> Select Vpc(same one we choose for Ec2) -> Mappings -> Select Az(Availability     Zones) -> create SG that has http/https and a custom Tcp with port range 9876 ->  Listeners and routing -> select Target group previosly created -> Create Load balancer

# Create ASG(Auto Scaling Group)

 - Auto scaling groups -> Create Auto Scaling group -> enter name of ASG -> Select Launch template previosly created -> select vpc(The one we used on Ec2) -> Select Availability zones -> Attach to an existing load balancer -> Select Load Balancer -> Health checks ->  Select Turn on ELB health checks -> Helath check grace period(120) -> next -> Select Group Size Recomended: Desired(2), Minimum(1), Maximum(3) -> Scaling Policies(none) -> Add Notification -> Create Topic -> Eneter topic name -> Add your email as recipients -> Choose event types(Launch, Terminate) make sure to confirm subscriptions -> Click next  until create ASG -> Create ASG


# Create ASG dynamic policy

- Slelect ASG -> Automatic Scaling -> Dynamic caling policies -> Create Dynamic scaling  policy -> Policy type -> Simple Scaling -> Enter policy name -> CLoudWatch alarm -> Create  CloudWatch 

- Metric -> Select Metric -> EC2 -> By ASG -> Select AutoScalingGRoupName to populate the ASG you just created -> select your specific ASG that has the metric Value of          "CPUUtilization" ->   Select Metric

- Specify Metric and conditions -> Static(average) -> Period(1minute) -> Condition -> Threshold(Satic) -> Whenever CPUtilization is (Greeater/Equal) -> Than(Enter Threshold) 

- Notifications -> Create topic -> Enter topic name -> Enter Email endpoint -> create Topic

- Name and description -> Enter alarm name -> Next -> Create Alarm

- Go back Create Dynamic Scaling Polocy -> CLick Refresh button next to CLoudWatch Alarm  and select Newly created alarm -> Take Action -> Add / 1 / capacity units -> And then  Wait (60) -> Create 

# Now you have just Deployed A Flask Application With Gunicorn and Nginx On a Ec2 With ELB,ASG allowing the application to have high scalability and resiliency
        

        


