# INSTALL AND CONFIGURE JENKINS SERVER
Step 1 – Install the Jenkins server <br>
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
- Install JDK (since Jenkins is a Java-based application)
`sudo apt update`
`sudo apt install default-jdk-headless`
- Install Jenkins
`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`<br>
`sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`<br>
`sudo apt update`<br>
`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA`
`sudo apt-get install jenkins`<br>
- Make sure Jenkins is up and running
`sudo systemctl status jenkins`<br>
** By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group **<br>

- Perform initial Jenkins setup.
- From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
- You will be prompted to provide a default admin password

- Retrieve it from your server:
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`<br>
