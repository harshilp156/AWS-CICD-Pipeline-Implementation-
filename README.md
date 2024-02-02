
# CICD Pipeline On AWS Mannaged Service

In this project I have implemented entire CICD pipeline on aws managed services.



## Code commit

Codecommit : For storing and managing my code I have used AWS code commit.

Now a root user cannot configure using ssh therefore we need a IAM user to configure and clone the repository into our local machine.
Also IAM user needs role to access the AWS Codecommit service.

To clone and commit into this repository we need a HTTPS git credentials for AWS Codecommit for this user. 


![Screenshot (543)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/0482f142-e05b-4927-85f2-5ff626ce0332)


## Code Build

Now for build my code I have used AWS Code Build service.

It works almost same as jenkins.

Source Provider : AWS Code Commit (Master Branch)

Environment : AWS codebild managed Image

Operating System : Ubuntu

Runttime: Standard

Image : Latest Version

Also to provide access of other services to the AWS codebild ,we need to give it a service role.


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

After the Code Build phase we need to have storage to save our build Artifacts.

For that I am using AWS S3(Simple Storage Service).

I have created a Bucket and configured the Artifact so that all my Artifact files can be stored in AWS S3 in Zip format.

![Screenshot (547)](https://github.com/harshilp156/AWS-CICD-Pipeline-Implementation-/assets/67538347/b296b171-4f8a-4324-81cd-3ce27f0eb707)


## Code Deploy

Now we need to creat an application to deploy our code on AWS code deploy service.

The application will use AWS EC2 instances to deploy my code.

Now after creating application , a deployment grout is needed.

Deployment Group : This is set of one or more servers where the application will be deployed.

Also Code deploy needs a service role to access other services (ec2, s3, codebuild ,etc..).

Also I will deploy this application on EC2 instance for that I need an EC2 instance.

## EC2 instance configurations

AMI : Ubuntu (Free Tier Eligible)

Instance Tyoe : t2.micro

Key Pair : To ssh

Default VPC

Security groups : 

Allow ssh traffic from : Anywhere

Allow HTTP traffic from internet

Allow HTTPS traffic from internet

Storage : 8gb

Also we need to configure the ec2 instance service role in order to give access of other services to this instance.


Now attached this EC2 instance with the deployment group.

Also we need a code deploy agent on our ec2 instance.

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

Now for continuous integration and continuous delivery for our application we need to create a code pipeline.

AWS CodePipeline is a continuous integration and continuous delivery (CI/CD) service provided by Amazon Web Services (AWS). Its primary purpose is to automate and streamline the process of releasing software changes. CodePipeline facilitates the delivery of code changes from source code repositories through various stages, such as build, test, and deployment, ultimately enabling a more efficient and reliable software release process.

We need a service role for code pipeline to provide access of other Services.

Source provider : AWS Codecommit

Code pipeline will continuously monitor for code change and release the changes after every commit to the main branch.

Build Provider : AWS Code Build

Deploy Provider : AWS Code Deploy

After the creation of the code pipeline it will automaticlly start the process of source -> Build -> Deploy.

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

