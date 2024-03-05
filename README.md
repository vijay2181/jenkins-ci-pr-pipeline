# jenkins-ci-pr-pipeline

## Install jenkins
```
INSTALLING JENKINS:
===================
- take t2.medium amazon linux 2 instance
- in real time just tell them that we are using Memory Optimized jenkins instances -> r5a.xlarge or r5a.2xlarge

- Install java:
sudo dnf install java-17-amazon-corretto-devel -y      ---> includes jdk
sudo dnf install java-17-amazon-corretto -y            ----> includes only jre

- Donwload Jenkins
use LTS version
https://pkg.jenkins.io/redhat-stable/

- add jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install jenkins

- when we install jenkins, it will create a service called jenkins
systemctl enable jenkins
systemctl start jenkins
systemctl status jenkins

- access jenkins
http://ip:8080


Install git
============
sudo yum install git

Install maven
=============
- we may have to install multiple maven verions, so on jenkins global tool configurations i will install maven
Dashboard > Manage Jenkins > Tools > Maven installations
Name: MAVEN3.9.5
Install automatically
Version: 3.9.5

Java17 project:
===============
- take java 17 project for maven build, because we have java17 installed for jenkins
https://github.com/vijay2181/java-maven-SampleWarApp.git

```
## PIPELINE SETUP:


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/479dc3c9-7d46-4fbc-ba25-94bd3bc572c3)


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/225c47d0-cb95-404f-b637-94115b072b16)

```
environment {
    PATH = "${tool 'MAVEN3.9.5'}/bin:$PATH"
}
```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/45550432-4ccd-49bf-8bfb-c644438dc60b)


```
(SONARQUBE) Add sonarqube details in pom.xml
============================================
- sonarqube tool is for code scanning, quality checking, publish qulaity report to sonarweb

Install sonarqube:
------------------
take t2.medium instance

sonarqube:
----------
sudo dnf install java-17-amazon-corretto-devel -y
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
sudo unzip sonarqube-10.4.1.88267.zip
- as a good security practice dont run sonarqube as root user, create a normal sonar user and provide him sudo privilages and run sonar with that user

sudo useradd sonar

visudo
sonar  ALL=(ALL)    NOPASSWD:ALL

sudo chown -R sonar:sonar sonarqube-10.4.1.88267
cd /opt/sonarqube-10.4.1.88267/bin/linux-x86-64

[ec2-user@ip-172-31-20-171 linux-x86-64]$ sh sonar.sh start
Starting SonarQube...
Failed to start SonarQube.

- because you are running sonar with ec2-user, so switch to sonar user

sudo su - sonar
cd /opt/sonarqube-10.4.1.88267/bin/linux-x86-64

[sonar@ip-172-31-20-171 linux-x86-64]$ sh sonar.sh start
Starting SonarQube...
Started SonarQube.
[sonar@ip-172-31-20-171 linux-x86-64]$ sh sonar.sh status
SonarQube is running (29383).

- enable 9000 port in SG 
- and access -> http://52.39.183.141:9000
- login -> username=admin, password=admin
- change admin password: admin123

if you get issues while bringing up sonar, check sonar.log es.log
if you run sonar as root and again start sonarqube with sonar user you will get issues
so Remove this folder and run again
rm -rf /opt/sonarqube-10.4.1.88267/temp


we need to install sonarqube on a seperate server and add sonarqube server details to pom.xml properties tag

---------------------------------------------------------------------
<properties>
      <sonar.host.url>http://35.165.197.226:9000</sonar.host.url>
      <sonar.login>admin</sonar.login>
      <sonar.password>admin123</sonar.password>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
---------------------------------------------------------------------

mvn sonar:sonar
```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/31d7ddf2-496f-4940-ab4c-90fbeb8c3fbe)


Jenkins Pipeleine:
------------------
```
pipeline {
    agent any
    environment {
        PATH = "${tool 'MAVEN3.9.5'}/bin:$PATH"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature-add-pom-17', url: 'https://github.com/vijay2181/java-maven-SampleWarApp.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }
        // Add more stages as needed
    }
}

```


