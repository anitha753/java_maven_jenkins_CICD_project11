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
   

5) Install Tomcat [ IN EC2 ]   
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
   77  sh startup.sh
   78  history
