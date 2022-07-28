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

- Log into Jenkins server as root user and install **docker engine** from this [official docker documentation](https://docs.docker.com/engine/install/#server)
   - Add the jenkins user into to docker group 
       ```sh 
       sudo -i 
       ```
       ```sh 
       id jenkins 
       ```
