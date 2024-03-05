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
```
