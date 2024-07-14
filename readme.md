## java-maven-app

See below the steps to deploy the application I toke.
- I then created a new repository on github and pushed the code to it.
- Work in jenkins vm using plugins Maven and do shared lib and some cred.
- With maven I build the jar file with the command ```mvn install```
- I then created a new droplet on digital ocean and installed java on it.
    - After setup of the droplet I enabled port 22 and 8080 in the firewall.(22 for ssh acess and 8080 for the app)
    - I now ssh into the droplet and used the following commands to update the droplet and install java
        - ``` sudo apt update```
        - ``` sudo apt upgrade -y```
        - ```sudo apt install openjdk-8-jre-headless htop nodejs docker net-tools -y```
- Then added my ssh keys from my laptop to also login by that user.
- Then i used scp to upload the jar file
    - scp target/java-maven-app-1.1.0-SNAPSHOT.jar ec2@{SERVER}:/root
- Then I logged into the droplet as jabbo and ran the jar file
    - ```java -jar java-maven-app-1.1.0-SNAPSHOT.jar```
- To verify the app was running I used curl to check the app
    - ```netstat -lpnt```
- After that I went to the server ip with the 8080 port and saw the app running. With the text Welcome to Java Maven
  Application