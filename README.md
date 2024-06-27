# jenkins-release-pipeline-deploying-to-kubernetes

![image](https://github.com/vijay2181/jenkins-release-pipeline/assets/66196388/31d371f0-b5cc-4772-afe1-d2a44fd9ce19)


we will get all branches from git repo and build the pipeline based on selected branch

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


SONARQUBE:
==========

```
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

### login to snarqube and create project plus token for external(jenkins,cli) authentication into sonarqube:


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/a159248c-64a0-43ad-8c57-9bc7f722f6d6)



- create a new project(local project) for our code analysis report publish, so to this project we can publish our reports

  

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/b3a55c50-50a6-4e51-a1ea-a8db12ebc020)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/e5839ab2-742a-4fe2-8b74-81ddacda24c5)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/60eabeb2-2caa-473e-958d-79eb15f87bec)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/6da085f2-26b8-46c8-bc28-45163eb2ad8f)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/81a563d9-d99c-4031-b98f-6a18780d53ee)


- for this test_project we will publish static code anaylsis report
- next we will create a token for authentication

### sonarqube token

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
login to jenkins master server

cd /opt/
sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
sudo unzip sonar-scanner-cli-5.0.1.3006-linux.zip

- now we need to inform sonarscanner where our sonarqube is running(server ip, port, token etc..), because sonarscanner can run independently, in sonar.properties file we need to mention

cd /opt/sonar-scanner-5.0.1.3006-linux/conf
sudo vi sonar.properties
-------------------------------------------------------------------------------------------------------
#----- Default SonarQube server
sonar.host.url=http://52.10.226.100:9000                	#this can be overriden at the command
sonar.token=sqa_1e72010ca912043557aa2bb7e37e3696f3333  		 #this can be overriden at the command
sonar.projectKey=test_project
sonar.projectname=test_project
sonar.sources=src/main/java          				#src location of project
sonar.java.binaries=target/classes   				#target location of project
--------------------------------------------------------------------------------------------------------

chmod +x /opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner
sudo chown -R jenkins:jenkins sonar-scanner-5.0.1.3006-linux/

- we need to goto project directory(/var/lib/jenkins/workspace/ci-pr) and run sonar scanner from there where (src/main/java) (target/classes ) folders are present
- sonar scanner location -> /opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner
- so here we are using generic sonar scanner which can be used for maven,gradle,ant etc....

/opt, /tmp directory are more permissive, allowing Jenkins to execute the SonarScanner CLI without encountering permission denied errors
The jenkins user typically has limited permissions to execute binaries in the home directory of other users, such as /home/ec2-user,

```

Jenkins Pipeleine:
------------------

you can find the official docs for sample sonar stage for jenkins pipeline

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/4c18f9f9-cde5-4002-ac5a-8930ed174aa6)


```
https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/jenkins-extension-sonarqube/
```


### confifigure sonarqube token inside jenkins server

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/81321f89-26fe-4eaf-a6f7-3b019869080f)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/2a293279-3130-4407-a4e8-29bbec3809c4)



### dynamically list all git branches inside jenkins pipeline

```
we need to dynamically list all git branches -> for this purpose we need to install Active Choices plugin, in that use -> Active Choices Reactive Parameter
https://medium.com/@g4b1s/dynamically-list-git-branches-in-jenkins-job-parameter-3e6e849f8a98

Active Choices Reactive Parameter
name: branches
groovy script:

def gettags = ("git ls-remote -t -h https://github.com/vijay2181/java-maven-SampleWarApp.git").execute()
return gettags.text.readLines().collect { 
  it.split()[1].replaceAll('refs/heads/', '').replaceAll('refs/tags/', '').replaceAll("\\^\\{\\}", '')
}

```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/139a4720-b8b5-43d1-a0b5-fd08feeb1d78)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/d83c2998-fae1-4129-9423-30ca796e1c37)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/9fb6bf34-a855-4ed5-b649-e2acdb7ea8ed)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/822edb4c-fce3-48c6-af02-b46e52622ce5)

- come down and apply
- we need to approve the script if necessary
- manage jenkins -> In process script approval -> approve the script
- go back to job and reload the page, you will get drop downs
- now execute below steps for git branches dynamic listing

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/90d3ba40-4027-4df3-a6bb-b23f82aa77ed)


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/8d962c74-6cfb-447d-ad93-b9e506a71c3e)





```
def gettags = ("git ls-remote -t -h https://github.com/vijay2181/java-maven-SampleWarApp.git").execute()
return gettags.text.readLines().collect { 
  it.split()[1].replaceAll('refs/heads/', '').replaceAll('refs/tags/', '').replaceAll("\\^\\{\\}", '')
}


pipeline {
    agent any

    stages {
        stage('print user selected branch'){
            steps{
                     echo "${params.branches}"
            }
        }
    }
}

```


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/225c47d0-cb95-404f-b637-94115b072b16)

```
environment {
    PATH = "${tool 'MAVEN3.9.5'}"
}
```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/45550432-4ccd-49bf-8bfb-c644438dc60b)


### Detailed Breakdown

```
Detailed Breakdown:
====================
Clean and package the project:
mvn clean package: This step compiles the source code, runs tests, and packages the code into a JAR or WAR file.
The important part is the compilation and test execution, which ensure the code is in a state that can be analyzed by SonarQube.

Run SonarQube analysis:
mvn sonar:sonar: This step does not directly analyze the JAR/WAR file but analyzes the source code and compiled
bytecode present in the target directory.
When you run mvn package, Maven places the compiled classes and other build artifacts in the target directory.
SonarQube analyzes these artifacts, not the final JAR/WAR package.

Deploy the project:
mvn deploy: This step uploads the JAR/WAR to a remote repository or server, assuming it passed the analysis.
```


#### Final Release Pipeline


```
pipeline {
    agent any
    
    environment {
        MAVEN = "${tool 'MAVEN3.9.5'}/bin/mvn"
        SONAR_HOST_URL = 'http://54.186.90.185:9000'
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_PROJECT_KEY = 'test_project'
        SONAR_PROJECT_NAME = 'test_project'
    }
    
    stages {

        stage('Print Selected Branch') {
            steps {
               echo "${params.branches}"
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${params.branches}", url: 'https://github.com/vijay2181/springboot-mongo-docker.git'
            }
        }
        
        stage('Build Maven Project') {
            steps {
                sh '$MAVEN clean package'
            }
        }
        
        stage('Run SonarScanner CLI on Maven project') {
            steps {
                sh '/opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner -Dsonar.host.url="${SONAR_HOST_URL}" -Dsonar.token="${SONAR_TOKEN}" '
            }
        }
        
        stage('Sonar Status Check') {
            steps {
                sh '''
                    #!/bin/bash
                    echo "This is a shell script within a Jenkins pipeline stage"
                    response=$(curl -u "${SONAR_TOKEN}": "${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}")
                    project_status=$(echo "$response" | jq -r '.projectStatus.status')
                    echo "Project Status: $project_status"
                    if [ "$project_status" == "OK" ]; then
                        echo "Project Status is OK."
                    else
                        echo "Project Status is ERROR. Exiting the pipeline"
                        exit 1
                    fi
                '''
            }
        }
       stage('Deploy to Next Stages') {
          steps {
             echo "This code can be deployable"
          }
       }       
    }
}


```


## Nexus Installation and Integration

```
- take t2.medium instance

- install java

sudo dnf install java-1.8.0-amazon-corretto-devel

[ec2-user@ip-172-31-16-247 ~]$ java -version
openjdk version "1.8.0_402"
OpenJDK Runtime Environment Corretto-8.402.08.1 (build 1.8.0_402-b08)
OpenJDK 64-Bit Server VM Corretto-8.402.08.1 (build 25.402-b08, mixed mode)

sudo wget -O nexus.tar.gz https://download.sonatype.com/nexus/3/latest-unix.tar.gz

sudo tar -xvf nexus.tar.gz

sudo mv nexus-3* nexus

sudo adduser nexus

visudo
nexus ALL=(ALL)   NOPASSWD:ALL

sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work

sudo vi /opt/nexus/bin/nexus.rc
#run_as_user="nexus"

#running nexus as service 
sudo ln -s /opt/nexus/bin/nexus /etc/init.d/nexus

sudo su - nexus

sudo systemctl enable nexus

sudo systemctl start nexus

sudo systemctl status nexus

```

- in pom.xml add the nexus server details
- before adding, we need to create two repos(release,snapshot) and add those repos urls in pom.xml
- now we need to push those jar/war files to nexus repository
- we need create two repositories(snapshots and releases) for a project to push artifacts to nexus when build is triggered
- nexus authentication details should be placed in maven settings.xml


![image](https://github.com/vijay2181/jenkins-release-pipeline/assets/66196388/9932b1ca-de1d-4e19-862c-288d6ef18670)


- depending upon pom.xml version (<version>0.0.1-SNAPSHOT</version>), war will be deployed in sanpshot or release repository of nexus
  
![image](https://github.com/vijay2181/jenkins-release-pipeline/assets/66196388/d8d106c5-12c5-4ce2-ad42-f7a61243cc88)

```
add below tag for nexus server url inside pom.xml, before adding we need to create snapshots and releases repos for project
the nexus id in settings.xml should be matched with pom.xml nexus id name

vi /home/ec2-user/java-maven-SampleWarApp/pom.xml
--------------------------------------------------
            <distributionManagement>
                        <snapshotRepository>
                                <id>nexus</id>
                                <url>http://54.245.43.112:8081/repository/app-snapshots/</url>
                        </snapshotRepository>
                        <repository>
                                <id>nexus</id>
                                <url>http://54.245.43.112:8081/repository/app-releases/</url>
                        </repository>
                </distributionManagement>
-------------------------------------------------

- add nexus credentials inside maven server settings.xml between below tags
- but we have not installed maven software on jenkins server manually, we have installed maven through global tool configuration
- /var/lib/jenkins/tools   --> inside this jenkins home directory we will get maven software which is installed through global tool configuration
- /var/lib/jenkins/tools/hudson.tasks.Maven_*/maven3.8.5/conf/settings.xml

vi /var/lib/jenkins/tools/hudson.tasks.Maven_*/maven3.8.5/conf/settings.xml
add content between servers tag
<servers>

</servers>
---------------------------
<servers>
   <server>
   <id>nexus</id>
   <username>admin</username>
   <password>admin</password>
   </server>
</servers>
------------------------------
- add nexus repo url details in  -> pom.xml
- add nexus repo credentails in  -> settings.xml


- to  upload package into remote repository, we need to use below maven command
mvn deploy


- like this war/jar versions will be created for only snapshot repos, for releases you wont get version sub tags
snashot repo
0.0.1-1, 0.0.1-2, 0.0.1-3  etc...

release repo
0.0.1, 0.0.2,0.0.3 etc...
- snapshot repos contain on going development packages
- production packages will be uploaded to release repos

my current version inside pom.xml is
<version>0.1-SNAPSHOT</version>
so artifacts will be only uploaded to snapshots repo

but when we have releases and i remove SNAPSHOT and change the version to below
<version>0.0.1</version>
mvn clean deploy
then artifacts will be only uploaded to release repo
- if i again run mvn deploy, then i will get error
Repository does not allow updating assets: vijay-releases (400)
- because it wont override existing version in release repo, so we need to chnage version tag to below and run
<version>0.0.2</version>
- the release versions will not get override in release repos
- we need to change any version number, then only it will allow to build
- but you can allow to redeploy same version if you enable the option for release repo to redeploy/override


```

- add below step in jenkinsfile

```
stage('Deploy WAR file to Nexus') {
            steps {
                sh '$MAVEN clean deploy'
            }
        }
```


```
pipeline {
    agent any
    
    environment {
        MAVEN = "${tool 'MAVEN3.9.5'}/bin/mvn"
        SONAR_HOST_URL = 'http://54.186.90.185:9000'
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_PROJECT_KEY = 'test_project'
        SONAR_PROJECT_NAME = 'test_project'
    }
    
    stages {

        stage('Print Selected Branch') {
            steps {
               echo "${params.branches}"
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${params.branches}", url: 'https://github.com/vijay2181/springboot-mongo-docker.git'
            }
        }
        
        stage('Build Maven Project') {
            steps {
                sh '$MAVEN clean package'
            }
        }
        
        stage('Run SonarScanner CLI on Maven project') {
            steps {
                sh '/opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner -Dsonar.host.url="${SONAR_HOST_URL}" -Dsonar.token="${SONAR_TOKEN}" '
            }
        }
        
        stage('Sonar Status Check') {
            steps {
                sh '''
                    #!/bin/bash
                    echo "This is a shell script within a Jenkins pipeline stage"
                    response=$(curl -u "${SONAR_TOKEN}": "${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}")
                    project_status=$(echo "$response" | jq -r '.projectStatus.status')
                    echo "Project Status: $project_status"
                    if [ "$project_status" == "OK" ]; then
                        echo "Project Status is OK."
                    else
                        echo "Project Status is ERROR. Exiting the pipeline"
                        exit 1
                    fi
                '''
            }
        }
		
        stage('Deploy WAR file to Nexus') {
            steps {
                sh '$MAVEN clean deploy'
            }
        }       
    }
}
```

## Setup Kubernetes Cluster:

- follow below link to setup k8s cluster using kubeadm

```
https://github.com/vijay2181/kubernetes-setup-using-kubeadm.git
```

## Build Stage

- cofigure dockerhub password inside jenkins credential manager using secret text

```
stage("Docker Push Image") {
            steps {
                withCredentials([string(credentialsId: 'Docker_Password', variable: 'Docker_Password')]) {
                    // Login to Docker Hub
                    sh "docker login -u vijay2181 -p ${Docker_Password}"
                }
                // Build Docker image
                sh "docker build -t vijay2181/springboot-mongo-docker:${env.BUILD_NUMBER} ."
                
                // Push Docker image to Docker Hub
                sh "docker push vijay2181/springboot-mongo-docker:${env.BUILD_NUMBER}"
            }
        }
```

## Install kubectl and add kubeconfig in Jenkins server

```
sudo -i

https://pwittrock.github.io/docs/tasks/tools/install-kubectl/

cd ~jenkins/
sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

sudo chown -R jenkins:jenkins kubectl
chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl

copy the config file from master to jenkins server, jenkins user home directory
root@k8smaster:~/.kube# cat config

sudo vi .kube/config
sudo chown -R jenkins: ~jenkins/.kube/

kubectl get nodes 

- add /etc/hosts file from k8s master to jenkins server
- also add below address details in /etc/hosts file of jenkins, because jenkins server dont know hostname of k8s master, incase you have added below addresses hosts file
- sudo vi /etc/hosts
172.31.26.79 k8smaster.example.net k8smaster
172.31.28.26 k8sworker1.example.net k8sworker1
172.31.21.107 k8sworker2.example.net k8sworker2

- jenkins user has permission to execute kubectl commands by jenkins jobs

- open 6443 port on jenkin server for kubectl and master communication
```


## Final Pipeline

```
pipeline {
    agent any
    
    environment {
        MAVEN = "${tool 'MAVEN3.9.5'}/bin/mvn"
        SONAR_HOST_URL = 'http://35.94.22.107:9000'
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_PROJECT_KEY = 'test_project'
        SONAR_PROJECT_NAME = 'test_project'
    }
    
    stages {

        stage('Print Selected Branch') {
            steps {
               echo "${params.branches}"
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${params.branches}", url: 'https://github.com/vijay2181/springboot-mongo-docker.git'
            }
        }
        
        stage('Build Maven Project') {
            steps {
                sh '$MAVEN clean package'
            }
        }
        
        stage('Run SonarScanner CLI on Maven project') {
            steps {
                sh '/opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner -Dsonar.host.url="${SONAR_HOST_URL}" -Dsonar.token="${SONAR_TOKEN}" '
            }
        }
        
        stage('Sonar Status Check') {
            steps {
                sh '''
                    #!/bin/bash
                    echo "This is a shell script within a Jenkins pipeline stage"
                    response=$(curl -u "${SONAR_TOKEN}": "${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}")
                    project_status=$(echo "$response" | jq -r '.projectStatus.status')
                    echo "Project Status: $project_status"
                    if [ "$project_status" == "OK" ]; then
                        echo "Project Status is OK."
                    else
                        echo "Project Status is ERROR, Exiting pipeline."
						exit 1
                    fi
                '''
            }
        }
        stage('Deploy WAR file to Nexus') {
            steps {
                sh '$MAVEN clean deploy'
            }
        }
        
        stage("Docker Push Image") {
            steps {
                withCredentials([string(credentialsId: 'Docker_Password', variable: 'Docker_Password')]) {
                    // Login to Docker Hub
                    sh "docker login -u vijay2181 -p ${Docker_Password}"
                }
                // Build Docker image
                sh "docker build -t vijay2181/springboot-mongo-docker:${env.BUILD_NUMBER} ."
                
                // Push Docker image to Docker Hub
                sh "docker push vijay2181/springboot-mongo-docker:${env.BUILD_NUMBER}"
            }
        }
		
        stage("Deploy on Kubernetes cluster") {
            steps {
                // Build Docker image
                sh "kubectl get nodes"
				sh "kubectl apply -f springBootMongo.yml"
				sh "sleep 10"
				sh "kubectl get svc"

            }
        }

    }
}

```



## Addon

- we can get latest version from nexus repository by using below script

```
#!/bin/bash

NEXUS_URL=http://35.85.147.67:8081
MAVEN_REPO=mongo-snapshots
GROUP_ID=com.mt
ARTIFACT_ID=spring-boot-mongo
VERSION=1.0-SNAPSHOT
FILE_EXTENSION=jar

# Function to extract the timestamp from the download URL
extract_timestamp() {
    url="$1"
    timestamp=$(echo "$url" | grep -oE '[0-9]{8}\.[0-9]{6}-[0-9]+')
    echo "$timestamp"
}

# Get the download URLs
urls=$(curl -s --user admin:admin -X GET "${NEXUS_URL}/service/rest/v1/search/assets?repository=${MAVEN_REPO}&maven.groupId=${GROUP_ID}&maven.artifactId=${ARTIFACT_ID}&maven.baseVersion=${VERSION}&maven.extension=${FILE_EXTENSION}" -H  "accept: application/json"  | jq -rc '.items | .[].downloadUrl')

# Initialize variables to hold the latest timestamp and URL
latest_timestamp=""
latest_url=""

# Loop through each URL
while IFS= read -r url; do
    timestamp=$(extract_timestamp "$url")
    # Compare timestamps
    if [[ -z $latest_timestamp || $timestamp > $latest_timestamp ]]; then
        latest_timestamp=$timestamp
        latest_url=$url
    fi
done <<< "$urls"

echo "Latest URL: $latest_url"

final_url=$latest_url' --http-user=admin --http-password=admin'

wget $final_url
```

- we can also use ansible to build docker image,tag,push and deploy
```
  https://github.com/vijay2181/jenkins-docker-ansible-deployment
```















