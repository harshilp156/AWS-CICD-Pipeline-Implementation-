
# AWS Managed CI/CD Pipeline

## Overview
This project showcases the implementation of a robust Continuous Integration and Continuous Deployment (CI/CD) pipeline utilizing AWS managed services. The pipeline seamlessly integrates AWS CodeCommit for version control, AWS CodeBuild for automated builds, AWS S3 for artifact storage, AWS CodeDeploy for application deployment on EC2 instances, and AWS CodePipeline to orchestrate the entire workflow.



## Code commit

For code storage and management, AWS CodeCommit is employed.

A root user is unable to configure using SSH, necessitating an IAM user for repository configuration and cloning.

The IAM user requires a role to access AWS CodeCommit.

HTTPS Git credentials are required for cloning and committing into the repository for this user.

![Screenshot (543)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/0482f142-e05b-4927-85f2-5ff626ce0332)


## Code Build

CodeBuild, an AWS service, is utilized for code building.

Functions similarly to Jenkins.

Source Provider: AWS CodeCommit (Master Branch).

Environment: AWS CodeBuild managed Image.

Operating System: Ubuntu.

Runtime: Standard.

Image: Latest Version.

A service role is required to provide access to other services for AWS CodeBuild.


![Screenshot (544)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/294b78e3-c69b-4b51-801a-2438b6aacef3)

## BuildSpec.yml Code

BuildSpec.yml is a YAML formatted text file (configuration file) which defines the series of build phases and commands required to build and test the project.

The Buildspec file includes information such as:

Phases: Defines the build phases, such as install, pre-build, build, post-build, etc. Each phase can contain a series of commands or actions to be executed.

Environment Variables: Specifies environment variables required during the build process.

Build Commands: Contains the actual commands that CodeBuild will execute during each phase of the build.


```bash
  version: 0.2

phases:
  install:
    commands:
      - echo Installing NGINX
      - sudo apt-get update
      - sudo apt-get install nginx -y
  build:
    commands:
      - echo Build started on `date`
      - cp index.html /var/www/html/
  post_build:
    commands:
      - echo Configuring NGINX

artifacts:
  files:
    - '**/*'


```
Here the CI (coninuous integration) part is complete.


## Storing Artifact

AWS S3 (Simple Storage Service) is used for storing build artifacts.

A bucket is created to store all artifact files in AWS S3 in Zip format.

![Screenshot (547)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/b296b171-4f8a-4324-81cd-3ce27f0eb707)


## Code Deploy

Application deployment on AWS CodeDeploy service is executed through AWS EC2 instances.

Deployment Group: A set of one or more servers where the application is deployed.

CodeDeploy requires a service role to access other services (EC2, S3, CodeBuild, etc.).

EC2 instance configurations include Ubuntu (Free Tier Eligible), t2.micro, SSH key pair, Default VPC, security group settings, and 8GB storage.

Also we need a code deploy agent on our ec2 instance.

A CodeDeploy agent is installed on the EC2 instance using a provided bash script.

AWS CodeDeploy agent :This is a software component that runs on the instances in our deployment group. It is a lightweight daemon process that facilitates the deployment of applications and updates on Amazon EC2 instances.The agent is a critical part of the AWS CodeDeploy service, enabling seamless and automated deployment workflows.

The AWS CodeDeploy agent:

Deployment Execution: Executes deployment instructions, communicates with AWS CodeDeploy to retrieve details, and reports deployment status.

Lifecycle Events: Progresses through events (e.g., ApplicationStop, BeforeInstall) allowing customization and tasks at different deployment stages.

Rollback Support: Facilitates rollbacks in case of deployment issues, ensuring application availability and stability.

Communication with AWS CodeDeploy: Interacts with AWS CodeDeploy service, pulling deployment details and reporting actions and status.

Updates and Upgrades: Receives updates from AWS to ensure compatibility, leverage new features, and maintain security.

## Here is the bash script for AWS code deploy Agent

Create a shell script with the below contents and run it.

```bash
#!/bin/bash 
```
```bash
# This installs the CodeDeploy agent and its prerequisites on Ubuntu 22.04.  
```
```bash
sudo apt-get update 
```
```bash
sudo apt-get install ruby-full ruby-webrick wget -y 
```
```bash
cd /tmp 
```
```bash
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb
```
```bash
mkdir codedeploy-agent_1.3.2-1902_ubuntu22 
```
```bash
dpkg-deb -R codedeploy-agent_1.3.2-1902_all.deb codedeploy-agent_1.3.2-1902_ubuntu22 
```
```bash
sed 's/Depends:.*/Depends:ruby3.0/' -i ./codedeploy-agent_1.3.2-1902_ubuntu22/DEBIAN/control 
```
```bash
dpkg-deb -b codedeploy-agent_1.3.2-1902_ubuntu22/ 
```
```bash
sudo dpkg -i codedeploy-agent_1.3.2-1902_ubuntu22.deb 
```
```bash
systemctl list-units --type=service | grep codedeploy 
```
```bash
sudo service codedeploy-agent status
```

## appspec.yml file

The appspec.yml (Application Specification) file is a crucial component in the AWS CodeDeploy deployment process. It's a YAML-formatted file that provides instructions to CodeDeploy on how to deploy and manage application during the deployment lifecycle. The appspec.yml file defines the deployment actions, hooks, and permissions needed to successfully deploy an application to your target environment.

The appspec.yml file will be run on AWS ec2 by code deploy agent.

```bash
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  AfterInstall:
    - location: scripts/install_nginx.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_nginx.sh
      timeout: 300
      runas: root
```

We need to add this file on root level of our project alonf with the folder containing th script files.

## Create Deployment

Now I have created the deployment.

At this point my continuous delivery part is completed.

![Screenshot (545)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/35b1f01d-cc81-4cfc-9b28-fdb487b1f501)

## Code Pipeline 

AWS CodePipeline, a CI/CD service, is utilized for continuous integration and continuous delivery.

A service role for CodePipeline is required for accessing other services.

Source Provider: AWS CodeCommit.

Build Provider: AWS CodeBuild.

Deploy Provider: AWS CodeDeploy.

The Code Pipeline is set to monitor code changes and release them after every commit to the main branch.

##Before the code changes


![Screenshot (546)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/cbd2f6fc-65d5-4f2e-8082-5c506bdf6f6c)

## After making change in index.html


![Screenshot (548)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/e8a5d064-c9cc-435e-9b1b-a08f3e84e753)

![Screenshot (549)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/2a2d62dd-03ff-4f9a-9fac-ef1e4bdcba36)

## My app is deployed on AWS EC2 instance with changes

![Screenshot (550)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/67feb8d7-46d8-4e5b-a006-531248723707)


## Tech Stack

**AWS:** IAM, EC2, S3,Artifact,  Codecommit,Codebuild, Codedeploy, Codepipeline

**HTML**

**CSS**

**VIM**

**GIT**

## Screenshots:

Screenshots are included at relevant stages to provide visual representation.

This project follows best practices for CI/CD implementation, ensuring an efficient and reliable software release process.

