CLICK ON RAW MODE TO SEE PROPER STEPS
---------------------------------------
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
vim jenkins.yml
---
- hosts: localhost
  connection: ssh

  tasks:
    - name: ensure system packages are updated
      ansible.builtin.dnf:
        name: "*"
        state: latest
        disable_gpg_check: false
    - name: install java
      ansible.builtin.dnf:
        name: java-21-amazon-corretto
        state: present
    - name: add jenkins repo
      ansible.builtin.get_url:
        url: https://pkg.jenkins.io/redhat-stable/jenkins.repo
        dest: /etc/yum.repos.d/jenkins.repo
        mode: '0644'
    - name: import jenkins gpg key
      ansible.builtin.rpm_key:
        state: present
        key: https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    - name: install jenkins
      ansible.builtin.dnf:
        name: jenkins
        state: present
     - name: start and enable jenkins service
      ansible.builtin.systemd_service:
        name: jenkins
        state: started
        enabled: true
        daemon_reload: true
    - name: wait for jenkins to startup
      ansible.builtin.wait_for:
        port: 8080
        host: {{inventory_hostname}}
        delay: 10
        timeout: 60
    - name: retrieve initial admin password
      ansible.builtin.shell: cat /var/lib/jenkins/secrets/initialAdminPassword
      register: output
      changed_when: false
    - name: display initial admin password
      ansible.builtin.debug:
          var: output.stdout
...
ansible-playbook jenkins.yml
++++++++++++++++++++++++++++++++++++++++++++++++++++

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

 ansible-playbook tomcat.yml --syntax-check
 ansible-playbook tomcat.yml --check
 ansible-playbook tomcat.yml
++++++++++++++
After tomcatsetup 
in browser publicip:8080 (port must be added in inbound rules)
username: tomcat  password: 123456
tomcat server setup successful
+++++++++++++++++
In master install GIT AND MAVEN
yum install git maven -y

 ++++++++++++++++++++
vim deployment.yaml
---
- hosts: dev
  connection: ssh

  tasks:
  - name: copying war file to tomcat server
    copy:
      src: /var/lib/jenkins/workspace/job1/target/myapp.war
      dest: /root/tomcat/webapps/

:wq  
++++++++++++++++++++++++++

IMPORTANT PLAYBOOKS
======================================

---
- hosts: dev
  connection: ssh

  tasks:
    - name: total memory of slave
      debug:
        msg: "ansible slave server memory is {{ansible_memory_mb.real}}"

    - name: slave server operating system
      debug:
        msg: "os family for slave {{ansible_fqdn}} is {{ansible_os_family}} and kernel is {{ansible_kernel}}"
    - name: get users from slave
      command: cat /etc/passwd
      register: output
    - name: users list of slve server
      debug:
        msg: "users in slave servers are {{output.stdout}}"
...

ansible-playbook one.yml
OUTPUT:
ok: [172.31.28.230]
TASK [total memory of slave] *****************************************************************************************************************************************
ok: [172.31.28.230] => {
    "msg": "ansible slave server memory is {'total': 961, 'used': 378, 'free': 583}"
}
TASK [slave server operating system] *********************************************************************************************************************************
ok: [172.31.28.230] => {
    "msg": "os family for slave ip-172-31-28-230.ec2.internal is RedHat and kernel is 6.1.161-183.298.amzn2023.x86_64"
}
TASK [get users from slave] ******************************************************************************************************************************************
changed: [172.31.28.230]
TASK [users list of slve server] *************************************************************************************************************************************
ok: [172.31.28.230] => {
    "msg": "users in slave servers are root:x:0:0:root:/root:/bin/bash\nbin:x:1:1:bin:/bin:/sbin/nologin\ndaemon:x:2:2:daemon:/sbin:/sbin/nologin\nadm:x:3:4:adm:/var/adm:/sbin/nologin\nlp:x:4:7:lp:/var/spool/lpd:/sbin/nologin\nsync:x:5:0:sync:/sbin:/bin/sync\nshutdown:x:6:0::FTP unprivileged user:/var/lib/stapunpriv:/sbin/nologin\nrpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin\ntcpdump:x:72:72::/:/sbin/nologin\nec2-user:x:1000:1000:EC2 Default User:/home/ec2-user:/bin/bash"
}
PLAY RECAP ***********************************************************************************************************************************************************
172.31.28.230              : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

========================
<<<install=present  uninstall=absent  update=latest restart=started>>>
SOME IMPORTANT COMMANDS ((HERE DEV MEANS HOSTS OF INVENTORY FILE))
ansible -m setup -a "filter=ansible_os_family" dev
ansible -m setup -a "filter=ansible_devices" dev
ansible dev -a "ls /home"
ansible dev -a "ls /root/"
ansible dev -b -m copy -a "src=one.yml dest=/root/"
ansible dev -b -m user -a "name=vinu state=present"

ansible dev -b -m yum -a "name=httpd state=present"
ansible dev -b -m service -a "name=httpd state=started"  
ansible dev -a "systemctl status httpd"
     |httpd.service - The Apache HTTP Server
     |Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     |Active: active (running) since Thu 2026-02-19 05:51:20 UTC; 1min 54s ago

vim index.html
<html>
   <head>
      <title>MYAPP</title>
   </head>
   <body bgcolor="pink">
      <marquee>WELCOME TO LINUX WORLD</marquee>
   </body>
</html>

ansible dev -b -m copy -a "src=index.html dest=/var/www/html/"
In Browser paste dev server public ip
you have successfully deployed your static app on httpd webserver.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++






   
