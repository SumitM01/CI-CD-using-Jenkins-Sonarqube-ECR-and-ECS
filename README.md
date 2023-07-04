# CI-CD-using-Jenkins-Sonarqube-ECR-and-ECS
This project configures an automated pipeline script using Jenkins which fetches the source code, builds it, scans the code using Sonarqube, uploads the artifact to Amazon ECR repository and finally deploys it on Amazon ECS.

## Overview
The key goals of continuous integration are to find and address bugs quicker, improve software quality, and reduce the time it takes to validate and release new software updates.

Looking in the past, developers on a team might work in isolation for an extended period of time and only merge their changes to the master branch once their work was completed. This made merging code changes difficult and time-consuming, and also resulted in bugs accumulating for a long time without correction. These factors made it harder to deliver updates to customers quickly.

Using **Continous Integration** helps us to tackle the issue by quickly building and running tests on the whole source code and quickly surfacing any errors for correction. **Continous Deployment** then helps us to quickly deploy the updated changes of the application without any delay.

In this project we are going to setup a code pipeline script using Jenkins which fetches the source code, builds it, scans the code using Sonarqube, uploads the artifact to Amazon ECR repository and finally deploys it on Amazon ECS. This project uses the source code of vprofile-application; a Java application intended to be a social media platform. 

## Tools/Services Used
- _Jenkins_ : A CI CD tool used worldwide for software development purposes.
- _Sonarqube Scanner_: A source code quality checking tool.
- _Maven_: Java build tool used worldwide to build artifacts from java source code.
- _Github_: A version control system and software repository known worldwide.
- _AWS EC2_: Provides virtual machines to host and run software on AWS cloud.
- *Elastic Container Registry(ECR)*: Fully managed container registry that makes it easy to store, manage, share, and deploy container images and artifacts anywhere. 
- *Elastic Container Service(ECS)*: Highly secure, reliable, and scalable way to run containers on AWS. 

## Project Architecture

[insert project architecure pic here]

The following processes take place when a code change is pushed to the central version control system (VSC):
- The source code is fetched from github(or any other VSC) by the git plugin installed inside jenkins.
- Then the jenkins runs the build job on the source code using Maven which builds the artifact.
- After that, unit tests are carried out by jenkins using maven plugin.
- After that, Jenkins pushes the code to sonarqube scanner for checkstyle analysis and sonar analysis of the code to detect the quality of the code.
- If the code passes the quality checks then, jenkins builds the artifact and uploads it into nexus repository ready for deployment.

## Implementation Details
### Launch EC2 instances each for jenkins, nexus and sonarqube
- Login to AWS management console.
- Go to EC2, launch instance.
- On local machine, open cmd, checkout to ci-jenkins branch on vprofile-project repository.
- Checkout to ci-jenkins branch.
- _Jenkins_
    - _OS_: ubuntu 20/18
    - _AMI_: t2.small
    - _Security group inbound rules_:8080 anywhere, 22 anywhere, 80 anywhere
    - _User data_: Contents of userdata/jenkins-setup.sh
- _Sonarqube_ :
    - _OS_: ubuntu 20/18
    - _AMI_: t2.medium
    - _Security group inbound rules_:9000 anywhere, 22 anywhere, 80 anywhere
    - _User data_: Contents of userdata/sonar-setup.sh

[insert img of launched ec2 instances here]

### Setup UI for Jenkins, nexus and Sonarqube
- **JENKINS** : get the password from the path provided and then create a new user.
- **SONARQUBE** : login using username=admin, password=admin.

### Install plugins on Jenkins
- On jenkins, go to _manage jenkins_ -> _manage plugins_ -> *available* then search and select following plugins:
    - Nexus artifact uplodaer
    - Sonarqube scanner
    - Pipeline maven integration
    - Build timestamp
    - Pipeline utility steps
    - Docker Pipeline
    - AWS SDK :: All
    - Cloudbees Docker build and publish
    - Amazon ECR
    - Pipeline : AWS steps
- Click on install without restart to install the plugins.

### Setup sonarqube scanner plugin on Jenkins

- __On Jenkins__
    - _manage jenkins_ -> *global tool configuration* -> *sonarqube scanner* -> *add* -> mention name and version and note it down.
    - *manage jenkins* -> *configure system* -> *sonarqube servers* -> *check environment variables* -> give name and url of the sonarqube instance and create.
- __On Sonarqube__
    - go to _my account_ -> _security_ -> generate token having jenkins as name and copy the secret generated key.
- __on Jenkins__
    - *manage jenkins* -> *configure system* -> *sonarqube servers* -> *server* *authentication token* -> select jenkins -> select kind as *secret text* and paste the secret text from token.

### Create a custom quality gate on Sonarqube server
- **On Sonarqube**
    - go to *quality gates* -> *create* -> parameter = *bugs* on *overall code* -> give condition as if bugs *greater than 60 then fail* .
    - *project* -> *settings* -> *quality gates* -> select the created quality gate.
    - go to *account* -> *webhooks* -> *create* -> give name and proper convention url for sonarqube webhook.

### Setup Docker Engine and AWS CLI on Jenkins
- **On Jenkins**
    - SSH to Jenkins instance and run the following commands to install docker in it.
    ```bash
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    - Add Jenkins user to Docker group using the following command
    ```bash
    chmod -a -G docker jenkins
    ```
    - Run the following command to install awscli:
    ```bash
    sudo apt-get upgrade
    sudo apt-get install awscli -y
    ```
    - Reboot the jenkins instance.

### AWS services setup
- __On AWS__
    - Search for IAM and navigate to IAM console.
    - Create a User with the following permissions:
        - AmazonEC2ContainerRegistryFullAccess
        - AmazonECS_FullAccess
    - Download the credentials file.
    - Create ECR repository
    - Create a ECS cluster
        - Create a task definition for the 
        - Create a service for the cluster using the task definition with load balancer, target group and security group.
    - After service has been created, edit the security group of load balancer to allow port 8080 from anywhere.

- __On Jenkins__
    - Go to _manage jenkins_->*Credentials*-> *SystemGlobal* -> *credentials (unrestricted)*-> add credentials by selecting type as *AWS credentials*

### Prepare the Jenkinsfile and run the pipeline
Prepare a pipeline as a code file to automate the entire setup using Jenkins. For reference use the uploaded pipeline code file.

## Conclusion 
Using the above steps I have setup an automated Continous Integration and Continous Deployment pipeline using Jenkins, Sonarqube, ECR and ECS for vprofile application. Use this document to implement the architecture and share among your friends. Have a good day.😊

## References 
- https://www.udemy.com/course/decodingdevops
- https://github.com/devopshydclub/vprofile-project/tree/docker
  
