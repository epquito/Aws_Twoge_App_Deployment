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

  1)Search 


  
  
