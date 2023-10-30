# Assesment-1
During this assessment we will learn how to properly deploy a Ec2 instance for a Flask application that will use both Gunicorn and NGINX. 

#Devleopment tools 

  1)AWS Account
  
  2)WSL2 or any linux op
  
  3)DBeaver or any db 

#Deployment steps

  1) Search Bar -> Ec2 -> Launch Instance
  
  2) Enter name -> Application and OS Images (SELECT AMAZON LINUX ) -> Amazon Machine Image(AMAZON LINUX 2 AMI) -> Instance Type(t2.micro) -> Key pair -> Select key pair     
     associated to your specific account

   3) Network Settings(CLICK EDIT) -> Vpc(CHOOSE EITHER DEFAULT VPC OR ANY OTHER ONE YOU MAY HAVE CREATED) -> Subnet (choose a public subnet) -> Auto-assign Ip(disable)
      Firewall/Security group -> Create Security Group -> Make security group that associates with the ec2 deployment -> Add Description -> Makes sure the security group has ->         SHH/Http/Https -> Source type(Custom) -> Source(0.0.0.0/0) -> Launch Instance


  
  
