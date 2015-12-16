# Linux-Server-Configuration
Udacity Project 5 - Linux Server Configuration project
Project Description

Installation of Ubuntu Linux on Amazon’s Elastic Compute Cloud (EC2) virtual machine to host  the Flask web application from Project 3: Item Catalog and make it accessible to the public.

The application is live at: [52.10.104.69][1] and [ec2-52-10-104-69.us-west-2.compute.amazonaws.com/][2]

##Steps to prepare the server

###Step 1: 
- Create new development environment at:
https://www.google.com/url?q=https://www.udacity.com/account%23!/development_environment&sa=D&usg=AFQjCNGv6S6CtUsCXf6gj9FkVtiDLeeq0A

- Download private key and move the private key file into the folder ~/.ssh 
 '$mv ~/Downloads/udacity_key.rsa ~/.ssh/'
- Restrict the key file to this account only (read+write)
 '$chmod 600 ~/.ssh/udacity_key.rsa'
-SSH into the instance
 '$ssh -i ~/.ssh/udacity_key.rsa root@52.10.104.69'

