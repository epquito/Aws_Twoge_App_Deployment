# Assesment-1
During this assessment we will learn how to properly deploy a Ec2 instance for a Flask application that will use both Gunicorn and NGINX. 

#Devleopment tools.

  1)AWS Account.
  
  2)WSL2 or any linux op.
  
  3)DBeaver or any db.

#Deploying Ec2.

  1) Search Bar -> Ec2 -> Launch Instance.
  
  2) Enter name -> Application and OS Images (SELECT AMAZON LINUX ) -> Amazon Machine Image(AMAZON LINUX 2 AMI) -> Instance Type(t2.micro) -> Key pair -> Select key pair     
     associated to your specific account.

   3) Network Settings(CLICK EDIT) -> Vpc(CHOOSE EITHER DEFAULT VPC OR ANY OTHER ONE YOU MAY HAVE CREATED) -> Subnet (choose a public subnet) -> Auto-assign Ip(disable)
      Firewall/Security group -> Create Security Group -> Make security group that associates with the ec2 deployment -> Add Description -> Makes sure the security group has ->         SHH/Http/Https -> Source type(Custom) -> Source(0.0.0.0/0) -> Launch Instance.

      
#Creatin Iam role for Ec2 to have Access to your S3 buckets

  1)  Search bar -> Iam -> Left side should be "Roles" and click -> Create role

  2)  Trusted entity type -> Aws service -> use case -> Ec2 -> next

  3)  Permission Polocies -> Search bar -> S3 -> AmazonS3FullAcces(click) -> Next

  4)  Role Details -> Role name -> Enter role name -> Create role

  5)  Search bar -> ec2 -> Choose Instance -> Click Actions -> Security -> Modify Iam role -> select the Iam role we just created 



#Deploying RDS

  1)  Search Bar -> RDS -> Create database

  2)  Standard create -> Engine type(PostgreSQL) -> Engine Version(15.3-r2) -> Templates(Free tier)

  3)  Settings -> DB instance identifier(Create and enter the name of db) -> Master username(Would reccomend changing username) -> Master Password(Enter personal password) ->           Confirm Password

  4)  Instance configuration -> db.t3.micro

  5)  Storage -> Leave default values -> Storage autoscaling -> deselect Enable Storage autoscaling

  6)  Connenctivity -> Don't connect to an Ec2 compute resource -> IPv4 -> Virtual private cloud(Choose the same vpc you selected on the Ec2) ->  DB subnet group(Leave 
      deafult) -> Public Access(Yes) -> VPC security group(Create new) -> New Vpc security group name(enetr name for security group)# leave the rest deafult -> Additional 
      configurations -> Database port(5432)

  7)  Database authentication -> Password authentication(Select)

  8)  leave all values under Monitoring as default and dont change anything

  9)  Additional configuration -> Database option -> Initial database name(Eneter database name associated to your flask app)

  10)  Select Create Database #Keep in mind the inilization of the database is going to take about 5-10 minutes

  11)  Once Database is created select and it should direct you to a page with a tab that has Connectivity & security

  12)  Select VPC Security groups -> select the Security group you created for your database -> Edit inbound rules  -> it should loom like this -> Security group rule Id ->           Type(PstgreSQL) -> Protocol(TCP) -> Port Rage(5432) -> Source(Custom) -> Destiniation(Security group of your EC2 instance previosly created)

#Creating a S3 bucket to hold static files

  1)  Search bar -> S3 -> Create bucket
    
  2)  Bucket name(Create bucket name) -> AWS Region(Use the Region assigned to you)

  3)  Object Ownershipp -> ACLs disabled

  4)  Block Public Acces settings for this bucket -> Block all public access(unclick) -> Acknowlege (click)

  5)  Bucket versioning(Enabled)

  6)  Create bucket(Leave everything else default)

  7)  Click on your bucket again

  8)  Object page -> Upload -> Choose the static files you want to upload

  9)  On tab click Permissions -> Create bucket policy That will allow Ec2 to have access to object/object version

  10)  {
      
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
      

#SSH into Ec2 to deploy Flask application With Gunicorn/Nginx

  1) Inside Aws console go to the search bar and type ec2

     
  2) Select a ec2 you want to ssh into

     
  3) once selected click on connect and click on ssh client

     
  4) There should be all the way on the bottom "Example:" that would have a ssh comand to copy

     
  5) go to your terminal

      
  6) make sure you change directory that holds your key pair you selected for that instance it should end with a .pem

      
  7) paste the command and it will ask a yes or no to continue type yes and press enter

      
  8) If the command for ssh  ends with a "root@ec2-ip" change the "root" to "ec2-user"

#Installing Dependicies and configuring Ec2 instance for deployment

  1) sudo yum update -y

  
  2) sudo yum install git -y


  3) sudo amazon-linux-extras install nginx1


  4) git clone https://github.com/chandradeoarya/twoge.git


  5) cd twoge
  
  
  6) sudo yum intall python3-pip -y
  
  
  7) python3 -m venv venv



