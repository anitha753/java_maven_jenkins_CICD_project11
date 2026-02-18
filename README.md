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


7)) install ansible and deploy an application on to tomcat server using ansible playbooks

3- servers  master slave1 slave2
same setup:
amazon-linux-2023 kernel, t2.micro, a keypair, 8GB storage

IN 3 SERVERS SAME SETUP
----------------------
sudo -i
useradd ansible
passwd ansible
12345678
12345678

visudo
   root    ALL=(ALL)       ALL
   ansible    ALL=(ALL)       NOPASSWD: ALL
ctrl+X ctrl+Y and Enter button to save and exit

vi /etc/ssh/sshd_config
PermitRootLogin yes     ----->38 line
PasswordAuthentication yes   ------->63 line
PermitEmptyPasswords no
:wq
systemctl start sshd
su - ansible

ONLY IN MASTER WHICH IS ANSIBLE SERVER
----------------------------------------
su - ansible
yum install python3 python3-pip python-devel openssl -y
pip3 install ansible --user
ansible --version

mkdir -p /etc/ansible
vi /etc/ansible/hosts
  [dev]
  private ip of slave1
  [test]
  private ip of slave2
:wq

ssh-keygen
ssh-copy-id ansible@privateip of slave
ssh ansible@privateip of slave
login successful
logout to comeout of the slave

ansible all --list-hosts
   displays conncted slaves
ansible dev -m ping 
   configured correctly or not ping-pong successful

SAME CONFIGURATION WITH ROOT USER WITHOUT ANSIBLE USER
--------------------------------------------------------
IN 3 SERVERS SAME SETUP
----------------------
sudo -i
passwd root
12345678
12345678

no need of this step
  |visudo
  |   root    ALL=(ALL)       ALL  
  |   ansible    ALL=(ALL)       NOPASSWD: ALL
  |ctrl+X ctrl+Y and Enter button to save and exit

vi /etc/ssh/sshd_config
#PermitRootLogin yes     ----->NO NEED
PasswordAuthentication yes   ------->63 line
PermitEmptyPasswords no
:wq
systemctl start sshd
su - ansible

ONLY IN MASTER WHICH IS ANSIBLE SERVER  
----------------------------------------
execute all below with in the ROOT USER ONLY 

su - i      
yum install python3 python3-pip python-devel openssl -y
pip3 install ansible --user
ansible --version

mkdir -p /etc/ansible
vi /etc/ansible/hosts
  [dev]
  private ip of slave1
  [test]
  private ip of slave2
:wq

ssh-keygen
ssh-copy-id root@privateip of slave
ssh root@privateip of slave
login successful
logout to comeout of the slave

ansible all --list-hosts
   displays conncted slaves
ansible dev -m ping 
   configured correctly or not ping-pong successful


++++++++++++++++++++++++++++++++++++++++++++++++++
install jenkins on master
install and configure tomcat on slave server using yaml file on master

vim tomcat.yml

---
- hosts: dev
  connection: ssh

  tasks:
    - name: install java
      ansible.builtin.dnf:
        name: java-17-amazon-corretto
        state: present

    - name: retrieve tar file from link
      ansible.builtin.get_url:
           url: https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.18/bin/apache-tomcat-11.0.18.tar.gz
           dest: "/root/"

    - name: untar the file
      command: tar -zxvf /root/apache-tomcat-11.0.18.tar.gz -C /root/

    - name: rename the tomcat file
      command: mv /root/apache-tomcat-11.0.18/ /root/tomcat

    - name: set roles in tomcat servers
      template:
        src: tomcat-users.xml
        dest: /root/tomcat/conf/tomcat-users.xml

    - name: modify context.xml to allow access
      template:
        src: context.xml
        dest: /root/tomcat/webapps/manager/META-INF/context.xml
    - name: create tomcat systemd service file
      copy:
        dest: /etc/systemd/system/tomcat.service
        content: |
          [Unit]
          Description=Apache Tomcat server
          After=network.target

          [Service]
          Type=forking
          User=root
          Group=root

          Environment="JAVA_HOME=/usr/lib/jvm/jre"
          Environment="CATALINA_HOME=/root/tomcat"
          ExecStart=/root/tomcat/bin/startup.sh
          ExecStop=/root/tomcat/bin/shutdown.sh
          Restart=on-failure

          [Install]
          WantedBy=multi-user.target

    -  name: reload system daemon
       systemd:
         daemon_reload: yes

    -  name: start and enable tomcat service
       ansible.builtin.service:
         name: tomcat
         state: started
         enabled: yes
 ...












   77  sh startup.sh
   78  history
