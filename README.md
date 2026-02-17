Here are the steps to do this:
Setup DOCKER

1.Log in to your AWS Console and go to EC2 > Security Groups.
2.Edit Inbound Rules and add a custom TCP rule for Port 8080 with Source 0.0.0.0/0


#-----------------------------------------------------------------------------------------------
- update your instance 
sudo dnf update -y

- Install and start Docker (if not already installed)
sudo dnf install docker -y
sudo systemctl start docker
sudo systemctl enable docker

- Pull the lightweight Alpine Jenkins image
- This is the smallest official LTS (Long Term Support) image
sudo docker pull jenkins/jenkins:lts-alpine

- Run the container
sudo docker run -d -p 8080:8080 --name jenkins-server jenkins/jenkins:lts-alpine
#-----------------------------------------------------------------------------------------------
Open your browser and enter: http://<YOUR_EC2_PUBLIC_IP>:8080
sudo docker logs jenkins-server
sudo docker ps -a --filter "name=jenkins-server"
sudo docker logs jenkins-server
#IF CONTAINER STOPED

- Update your existing container with a restart policy
sudo docker update --restart unless-stopped jenkins-server

WAR/EAR files
target/*.war

Context path
myweb



Setup TOMCAT
create a Docker file to create a tomcat server with creating password and access to manager
#-----------------------------------------------------------------------------------------------
FROM tomcat:9.0-jdk17-corretto-al2
- 1. Setting up the username, password, and roles in tomcat-users.xml
- This appends both manager-gui (for browser) and manager-script (for Jenkins) roles
RUN sed -i '/<\/tomcat-users>/i \
<role rolename="manager-gui"/>\n\
<role rolename="manager-script"/>\n\
<user username="admin" password="YourPassword123" roles="manager-gui,manager-script"/>' \
/usr/local/tomcat/conf/tomcat-users.xml
- 2. Disable the RemoteAddrValve to allow remote access
RUN sed -i 's/<Valve//g' /usr/local/tomcat/webapps/manager/META-INF/context.xml
- Optional: Set the working directory
WORKDIR /usr/local/tomcat/webapps/
#-----------------------------------------------------------------------------------------------
Steps to do after
Updated Deployment Workflow
Build your image: sudo docker build -t my-custom-tomcat .
Run it: sudo docker run -d -p 8081:8080 --name tomcat-manager-server my-custom-tomcat
Jenkins Integration: In your Jenkins "Deploy to container" plugin, you can now use the credentials:
Username: admin
Password: YourPassword123
Tomcat URL: http://<EC2_IP>:8081





- Syntax: docker cp <Source_Path_on_EC2> <Container_Name>:<Destination_Path_Inside_Container>
sudo docker cp /var/lib/docker/volumes/jenkins-data/_data/workspace/my-app/target/my-app.war tomcat-manager-server:/usr/local/tomcat/webapps/


IF REQUIRE:**
Automate this in your Jenkins Job**
To ensure you don't get errors or have to run manual commands every time, add this script to your Jenkins "Execute Shell" block:
Open your Jenkins Job > Configure.
Scroll down to Build Steps > Add build step > Execute shell.
Paste this script:

- 1. Identify the war file
WAR_FILE=$(find . -name "*.war" | head -n 1)
- 2. Check if file exists to avoid errors
if [ -z "$WAR_FILE" ]; then
    echo "Error: WAR file not found!"
    exit 1
fi
- 3. Deploy to Tomcat (This requires the Jenkins user to have sudo/docker permissions)
- If your Jenkins is in Docker, this command must be run on the HOST or via a remote trigger.
sudo docker cp $WAR_FILE tomcat-manager-server:/usr/local/tomcat/webapps/
********
