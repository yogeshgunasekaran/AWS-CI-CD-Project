# AWS-CI-CD-Project
This project is to deploy a web-application using AWS CI/CD Pipeline completely by only using the AWS services - CodeCommit, CodeBuild, CodeDeploy, Elastic Beanstalk (PaaS), S3 and RDS 

### Project Architecture
![aws pipeline drawio (1)](https://user-images.githubusercontent.com/106590073/181558212-91c9209f-4dc1-4970-9f51-eb34c2ae3f9e.jpg)

### Steps 
> 1. Create Elastic Beanstalk (PaaS) for the Application
> 2. Setup RDS Instance & Application Setup on Beanstalk
> 3. Code Commit
> 4. Code Build
> 5. Build & Deploy
> 6. Code Pipeline 

### Procedure

**Log into AWS,** <br>
- In **Elastic Beanstalk &rarr; Create Application &rarr; Add users** 
  - Give Application name as **beanstalk-app-env** and its **tags**. Choose the required **Platform** and its **version** for the application to run. Intially, keep Application code section with **sample application**, later we can deploy our artifact
  - Click **Configure more options** and configure the environment settings for the application as per the requirement. In **Capacity** &rarr; **Edit**, choose the Auto Scaling Group Environment type as **Load balanced** with **Min- 2** and **Max- 8** or as per the requirement. Choose the required **Instance type** and keep the **Availability Zones** as **any** and select all the AZ's in Placement section for high availability. Scaling triggers Metric choose **NetworkOut** or **CPUUtilization** as per the choice and **Save**. In **Rolling updates and deployments** &rarr; **Edit**, choose the **Deployment policy** as per the requirement, say for eg. **Rolling** with **Batch size 25%** and **Save**. In **Security** &rarr; **Edit**, generate a new key-pair for Beanstalk in EC2 service and attach it here. But, note: this key-pair should only be used as a bastion host to connect with the RDS server in backend for intiating the database for once. In **Network** &rarr; **Edit**, provision the Beanstalk with the project VPC network. Remaining settings can be tweaked as per the requirements. Once done with the environment settings configurations finally click **Create app**, in sometime this will create an environment in Beanstalk and run the sample code application on it
  - #### <ins> *Note* </ins>  : For Production purpose do not create an RDS service in the Beanstalk platform itself. Since, it gets deleted along when Beanstalk is been deleted
  
 - In **RDS &rarr; Create database** 
   - Choose **Standard create**, engine type **MySQL** and version **5.6.34**, template as **Free tier**. Give an DB instance identifier name, keep Master username as **admin** and Auto generate a password. Keep the remaining settings default as for the free tier. Create a new security group **beanstalk-RDS-SG**. In Additional configuration section, give Initial database name as **accounts**. This 'accounts' database will be used to setup the DB schema. Keep others defaults and finally click **Create database**. Click **View credential details** on the top right and store the credentials
   
 - In **EC2 &rarr;** configure the **Security Groups** 
   - Click the Instances that has been created by the Beanstalk and edit its inbound rules, **change &rarr;** SSH port22 from source: anywhere to **SSH port22 from source: My IP**
   - Click the RDS **'beanstalk-RDS-SG'** Security group and edit its inbound rules, **change &rarr;** MYSQL port3306 from source: My IP to **MYSQL port3306 from source: beanstalk-app-env-SG**
