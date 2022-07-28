# AWS-CI-CD-Project
This project is to deploy a web-application using AWS CI/CD Pipeline completely by only using the AWS services - CodeCommit, CodeBuild, CodeDeploy, Elastic Beanstalk (PaaS), S3 and RDS 

### Project Architecture
![aws pipeline drawio (1)](https://user-images.githubusercontent.com/106590073/181558212-91c9209f-4dc1-4970-9f51-eb34c2ae3f9e.jpg)

### Steps 
> 1. Elastic Beanstalk
> 2. RDS & Application Setup on Beanstalk
> 3. Code Commit
> 4. Code Build
> 5. Build & Deploy
> 6. Code Pipeline 

### Procedure

**Log into AWS,** <br>
- In **Elastic Beanstalk &rarr; Create Application &rarr; Add users** 
  - Give a **Application name** and its **tags**. Choose the required **Platform** and its **version** for the application to run. Intially, keep Application code section with **sample application**, later we can deploy our artifact
  - Click **Configure more options** and configure the environment settings for the application as per the requirement. In **Capacity** &rarr; **Edit**, choose the Auto Scaling Group Environment type as **Load balanced** with **Min- 2** and **Max- 8** or as per the requirement. Choose the required **Instance type** and keep the **Availability Zones** as **any** and select all the AZ's in Placement section for high availability. Scaling triggers Metric choose **NetworkOut** or **CPUUtilization** as per the choice and **Save**. In **Rolling updates and deployments** &rarr; **Edit**, choose the **Deployment policy** as per the requirement, say for eg. **Rolling** with **Batch size 25%** and **Save**.
 
