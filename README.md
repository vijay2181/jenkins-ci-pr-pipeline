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


you can find the official docs for sample sonar stage for jenkins pipeline

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/4c18f9f9-cde5-4002-ac5a-8930ed174aa6)


```
https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/jenkins-extension-sonarqube/
```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/479dc3c9-7d46-4fbc-ba25-94bd3bc572c3)


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/225c47d0-cb95-404f-b637-94115b072b16)

```
environment {
    PATH = "${tool 'MAVEN3.9.5'}/bin:$PATH"
}
```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/45550432-4ccd-49bf-8bfb-c644438dc60b)


```
(SONARQUBE) :
============
- sonarqube tool is for code scanning, quality checking, publish qulaity report to sonarweb
- we need to install sonarqube on a seperate server

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

```

login to snarqube and create project plus token for external(jenkins,cli) authentication into sonarqube:
=========================================================================================================

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/a159248c-64a0-43ad-8c57-9bc7f722f6d6)

- create a new project(local project) for our code analysis report publish, so to this project we can publish our reports

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/b3a55c50-50a6-4e51-a1ea-a8db12ebc020)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/e5839ab2-742a-4fe2-8b74-81ddacda24c5)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/60eabeb2-2caa-473e-958d-79eb15f87bec)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/6da085f2-26b8-46c8-bc28-45163eb2ad8f)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/81a563d9-d99c-4031-b98f-6a18780d53ee)

- for this test_project we will publish static code anaylsis report
- next we will create a token for authentication

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/cfab7a44-2318-4db5-8fb6-02fea7d1b12c)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/1f969c55-0b4a-42ef-8942-1ad3b827337f)

- generate token and copy it


Sonar Scanner Options:
======================

```
SonarQube scanning can be performed using three approaches:

1. SonarQube Scanner CLI: This involves using the SonarQube Scanner command-line interface. Configuration details such as SonarQube server URL, login credentials, etc., are specified in a `sonar.properties` file.
   
2. Jenkins SonarQube Scanner Plugin: Integration with Jenkins can be achieved using the SonarQube Scanner plugin. This allows for seamless integration with Jenkins pipelines, automating the analysis process.
   
3. Maven SonarQube Plugin: For Maven projects, SonarQube analysis can be initiated using the `mvn sonar:sonar` command. Configuration details are typically specified in the `pom.xml` file within the `<properties>` section.

Example Configuration in `pom.xml`:
------------------------------------
<properties>
    <sonar.host.url>http://35.165.197.226:9000</sonar.host.url>
    <sonar.login>admin</sonar.login>
    <sonar.password>admin123</sonar.password>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>


using the 'mvn sonar:sonar' command is a valid approach, it's generally recommended to use the SonarQube Scanner for Maven plugin (sonar-maven-plugin) for more comprehensive integration with Maven. This plugin provides more control and flexibility over the analysis configuration within the Maven build lifecycle.

- While using the SonarQube Scanner CLI can indeed provide a generic approach that is not tied to a specific build tool, it may require additional setup and configuration outside of the project's build scripts. This can introduce complexities, especially in a CI/CD environment where you might want seamless integration with your existing build tool.
The choice between using the SonarQube Scanner CLI versus the specific integrations like the Maven plugin or Jenkins plugin depends on your project's needs, existing infrastructure, and preferences for tooling and automation.

- but to be generic like if build tool is maven, gradle,ant etc... so to support all, i will go with sonarqube scanner cli package

- we need to install this, so im going to install this on same jenkins server cli


```

install sonarcli scanner on jenkis server itself
-------------------------------------------------

```
cd /tmp/
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
unzip sonar-scanner-cli-5.0.1.3006-linux.zip

- now we need to inform sonarscanner where our sonarqube is running(server ip, port, token etc..), because sonarscanner can run independently, in sonar.properties file we need to mention

cd /tmp/sonar-scanner-5.0.1.3006-linux/conf
vi sonar.properties
#----- Default SonarQube server
sonar.host.url=http://52.10.226.100:9000
sonar.token=sqa_1e72010ca912043557aa2bb7e37e3696f3333
sonar.projectKey=test_project
sonar.projectname=test_project
sonar.sources=src/main/java
sonar.java.binaries=target/classes


- we need to goto project directory and run sonar scanner from there (src/main/java)
- sonar scanner location -> /tmp/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner
- so here we are using generic sonar scanner which can be used for maven,gradle,ant etc....

/tmp directory are more permissive, allowing Jenkins to execute the SonarScanner CLI without encountering permission denied errors

```

Jenkins Pipeleine:
------------------

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/31d7ddf2-496f-4940-ab4c-90fbeb8c3fbe)


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


