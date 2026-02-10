1) Install Git [ IN EC2 ]
yum install git -y 
git --version
github repo: https://github.com/anitha753/java_maven_jenkins_CICD_project11.git
git commands:
   git remote add origin https://github.com/anitha753/java_maven_jenkins_CICD_project11.git
   git branch
   git branch -M main   (to change master branch to main in local repo)
   git branch
   git push -u origin main
   git log
   history

3) Install Java [ IN EC2 ]
yum install java
java –version
sudo yum install java-21-amazon-corretto-devel
sudo yum install java-21-amazon-corretto
sudo update-alternatives --config java
sudo wget -O /etc/yum.repos.d/jenkins.repo     https://pkg.jenkins.io/rpm-stable/jenkins.repo
sudo yum upgrade
sudo yum install jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
cat /var/lib/jenkins/secrets/initialAdminPassword

4) Install Maven [ IN EC2 ]   
   yum install maven -y   (JAVA17 no problem)
   java --version   (java 21)
   

5) Install Tomcat [ IN EC2 ]   (which supports java17 and 21)
cd /opt
  wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.18/bin/apache-tomcat-11.0.18.tar.gz
  tar -zxvf apache-tomcat-11.0.18.tar.gz
  mv apache-tomcat-11.0.18/ tomcat
cd tomcat/
  cd webapps/manager/META-INF/
    vi context.xml   --->  edit and save
  cd ../../../
  cd conf/
     vi tomcat-users.xml   --->  edit and save 
     vi server.xml      --->  port number edit and save
  cd ../
  cd bin/
     sh shutdown.sh
     sh startup.sh
====================================================================
6)Install sonarqube [IN EC2] which supports java17

[root@sonar~]# cat sonar.sh
#! /bin/bash
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
sudo yum install java-17-amazon-corretto -y
sudo update-alternatives --config java
mv sonarqube-8.9.6.50800 sonarqube
useradd sonar
chown -R sonar:sonar sonarqube
chmod 777 sonarqube
su - sonar

# use the below command manually after installation
#sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start
#echo "user=admin & password=admin"
[root@sonar~]#

============OR======
Create service for Sonarqube

sudo vim /etc/systemd/system/sonar.service

Paste the below into the file

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
Start Sonarqube and Enable service

sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
sudo tail -f /opt/sonarqube/logs/sonar.log

========================================================================================











   77  sh startup.sh
   78  history
