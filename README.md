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

#### **Log into AWS,** <br>
- In **Elastic Beanstalk &rarr; Create Application &rarr; Add users** 
  - Give Application name as **beanstalk-app-env** and its **tags**. Choose the required **Platform** and its **version** for the application to run. Intially, keep Application code section with **sample application**, later we can deploy our artifact
  - Click **Configure more options** and configure the environment settings for the application as per the requirement. In **Capacity** &rarr; **Edit**, choose the Auto Scaling Group Environment type as **Load balanced** with **Min- 2** and **Max- 8** or as per the requirement. Choose the required **Instance type** and keep the **Availability Zones** as **any** and select all the AZ's in Placement section for high availability. Scaling triggers Metric choose **NetworkOut** or **CPUUtilization** as per the choice and **Save**. In **Rolling updates and deployments** &rarr; **Edit**, choose the **Deployment policy** as per the requirement, say for eg. **Rolling** with **Batch size 25%** and **Save**. In **Security** &rarr; **Edit**, generate a new key-pair for Beanstalk in EC2 service and attach it here. But, note: this key-pair should only be used as a bastion host to connect with the RDS server in backend for intiating the database for once. In **Network** &rarr; **Edit**, provision the Beanstalk with the project VPC network. Remaining settings can be tweaked as per the requirements. Once done with the environment settings configurations finally click **Create app**, in sometime this will create an environment in Beanstalk and run the sample code application on it
  - #### <ins> *Note* </ins>  : For Production purpose do not create an RDS service in the Beanstalk platform itself. Since, it gets deleted along when Beanstalk is been deleted
  
 - In **RDS &rarr; Create database** 
   - Choose **Standard create**, engine type **MySQL** and version **5.6.34**, template as **Free tier**. Give an DB instance identifier name, keep Master username as **admin** and Auto generate a password. Keep the remaining settings default as for the free tier. Create a new security group **beanstalk-RDS-SG**. In Additional configuration section, give Initial database name as **accounts**. This 'accounts' database will be used to setup the DB schema. Keep others defaults and finally click **Create database**. Click **View credential details** on the top right and store the credentials
   
 - In **EC2 &rarr;** configure the **Security Groups** 
   - Click the Instances that has been created by the Beanstalk and edit its inbound rules, **change &rarr;** SSH port22 from source: anywhere to **SSH port22 from source: My IP**
   - Click the RDS **'beanstalk-RDS-SG'** Security group and edit its inbound rules, **change &rarr;** MYSQL port3306 from source: My IP to **MYSQL port3306 from source: beanstalk-app-env-SG**

 - **Deploy** the application **DB schema** into the RDS 'acounts' database
   - SSH to an Beanstalk Instance as a root user `sudo -i` and, 
    - Install the required packages `yum install git mysql -y`, clone the source code that contains the schema **'db_backup.sql'** file
    - Verify the connection to the RDS with `mysql -h <RDS-endpoint-here> -u admin -p<admin-pass-here> accounts`. Verify the connection and exit
    - Now, deploy the schema to the accounts database, `mysql -h <RDS-endpoint-here> -u admin -p<admin-pass-here> accounts < src/main/resources/db_backup.sql`
      
 #### **Manual Method** to Build and Deploy the Artifact from local system
   - First update the **applications.properties** file in `vim src/main/resources/applications.properties`. Edit and update **jdbc.url** with the RDS endpoint and **jdbc.username and jdbc.password** with admin and admin-pass
   - Now build the application Artifact, `mvn install`
   - Copy the Artifact to the desktop, `cp target/vprofile-v2.war ~/Desktop/`
   - In **Elastic Beanstalk &rarr; Environments &rarr;** click **App-env &rarr;** click **Upload**. Give a Version label **app-test** and upload the Artifact which is in the desktop.
   - Now, we can see the Artifact in the **Elastic Beanstalk &rarr; Application versions**. Select it **app-test**, in Actions **&rarr; Deploy**. Events can be seen in the **Environments**. One important step to be followed after this is to configure the load balancer **health check** for the application
   - In **Elastic Beanstalk &rarr; Environments &rarr; app-env** click the **Configuration**. In the **Load balancer** section **&rarr; Edit**. Select the **Processes** and the Health check Path from **'/' to '/login'**. Also, checkmark the **Stickiness policy enabled** and **Save** and click **Apply** at the bottom.
   - To make the application configuration changes to live, rollback to downgrade the app first with sample-app '**Elastic Beanstalk &rarr; Application versions**' select the **Sample Application &rarr; Actions &rarr; Deploy**. Sample application web-page will be loaded in sommetime. Now, again rollback our application '**Elastic Beanstalk &rarr; Application versions**' select the **App-env &rarr; Actions &rarr; Deploy**  

#### **Automation Method using AWS CI/CD Pipeline** 
##### <ins> *Note* </ins>  : Choose a region which has **'CodeArtifact'** in it
 - In **AWS CodeCommit &rarr; Repositories &rarr; Create repository** 
   - Give a repository name as **code-repo** and click **Create**. This will create a repository just like in GitHub repository.
 - In **IAM &rarr; Users &rarr; Add user**. Give user name as **code-admin**. Check mark the **Programmatic access** and click **Next:Permissions**. Choose **Attach existing policies directly &rarr; Create policy**. In Service, select **CodeCommit  &rarr; All CodeCommit actions**. In **Resources &rarr; Specific &rarr; AddARN** give the repo-region and repo-name and **Add**. Then, click **Review policy** and give policy a name and click **Create policy**. Now, attach this policy to the user **code-admin** and click **Create user**
 - In local machine, generate an SSH key-pair using `ssh-keygen.exe` with name as `/c/Users/yogesh/.ssh/codecommit_rsa`
 - In **IAM &rarr; Users &rarr; code-admin &rarr; Security-credentials**. In Access keys section, **Delete** the Access key. In **SSH keys for AWS CodeCommit** Upload SSH public key. In local machine, go to the path that contains the ssh key-pair `cd .ssh`, use `cat codecommit_rsa.pub` to copy and paste it in the **Upload SSH public key** in AWS and note the **SSH key ID**
 - In local machine, go to the path `cd /c/Users/yogesh/.ssh/` and create a **config** file `vim config` 
   
   > Host git-codecommit.*.amazonaws.com <br> 
   > &nbsp; &nbsp;  User < aws-SSH-key-ID-here > <br>
   > &nbsp; &nbsp;  IdentityFile < path-to-the-private-key-here > <br>
   
   Example,
    ```sh
    Host git-codecommit.*.amazonaws.com
      User APKAXIXFJTQEW2ZCTWED
      IdentityFile ~/.ssh/codecommit_rsa
    ```
     This **ssh_config_file** does the authentication when using the **codecommit** service which contains the public SSH key ID and private key.
     Also, change the file permission to executable 
     ```sh
     chmod 600 config
     ```
     To verify the authentication with the credentials provided use,
     ```sh
     ssh git-codecommit.us-east-2.amazonaws.com
     ```
