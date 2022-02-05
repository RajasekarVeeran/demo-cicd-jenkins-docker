# demo-cicd-jenkins-docker

This demo will help you to understand the basics of Jenkins and Docker and, their pivotal role in CI/CD. 

## Pre-Requisites:
- You need a Linux machine(VM/Server) with Docker installed.
  There are plenty of playgrounds online. Try these sites.
  - https://labs.play-with-docker.com/
  - https://www.katacoda.com/courses/docker/playground

## Let's play!
### Step 1: Install docker-compose
- Run this command to download the current stable release of Docker Compose:
  ```
  $ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  ```
- Apply executable permissions to the binary:
  ```
  $ sudo chmod +x /usr/local/bin/docker-compose
  ```

Note: Remove sudo if you are a root user

For more details: https://docs.docker.com/compose/install/

### Step 2: Install and run jenkins as a docker container
Run the below shell commands to setup directory structure for jenkins.
```
$ mkdir -p ~/jenkins-data/jenkins_home
$ chown 1000:1000 ~/jenkins-data/jenkins_home
$ cd ~/jenkins-data
$ touch docker-compose.yaml
```

Copy the below content to the docker-compose file created and save it.
```
version: '3'

services:
  jenkins:
    image: jenkins/jenkins
    environment:
      HTTP_PROXY: ${HTTP_PROXY}
      HTTPS_PROXY: ${HTTPS_PROXY}
    ports:
      - "8080:8080"
    volumes:
      - "./jenkins_home:/var/jenkins_home"
    networks:
      - net

networks:
  net:
```

Make sure you are in the jenkins-data directory where compose file is located and run the below command to create and start the jenkins container.
The option -d in this command starts the container in detached mode(background process).
```
$ docker-compose up -d
```
Run the below commands to see the downloaded images and running containers details.
```
$ docker image ls
$ docker ps
```
![image](https://user-images.githubusercontent.com/14373178/152638896-d42ef71d-0e9d-42ed-aac1-191701eac3f0.png)

If the ```docker ps``` command returns no result, the container might have failed to run.
Check the logs of the container with below command and troubleshoot it accordingly.
You can get the CONTAINER_ID from the ```docker ps -a``` command where the option -a shows all containers even the stopped ones.
```
$ docker logs <CONTAINER_ID>
```

### Step 3: Setup Jenkins
Once the jenkins container is up and running. Open this url in your browser to access the Jenkins UI.
```http://<Server-dns>:8080``` 
[OR] 
```http://<Server-ip>:8080```
You will see a page asking for initialAdminPassword. The password can be found in the docker logs output from the above command.
![image](https://user-images.githubusercontent.com/14373178/152643161-604e3a39-f8cd-4c77-9471-782bfd0b5bb8.png)

Next is a plugin installation page. If your server is behind a corporate proxy the plugin installations will fail eventually. So, select Custom Plugin selection option and select none and proceed to the next page.

Next page will ask you for username, password and fullname. Enter the details and thats it. You are now sucessfully redirected to the Jenkins Homepage.

Before beginning to play with jenkins, let us do some configurations to make things smoother.
1. Enable proxy compatibility to prevent '403 no valid crumb' error.
   ![image](https://user-images.githubusercontent.com/14373178/152643707-2c2a9bf7-c00a-4105-acac-5c6fdb5ef99e.png)

   Goto Manage Jenkins -> Configure Global Security -> CSRF Protection -> Check the option 'Enable proxy compatibility'
   ![image](https://user-images.githubusercontent.com/14373178/152643731-fb6f636a-c750-48b2-869f-d9803d31cb45.png)

2. Setup proxy configuration for plugin installation if your server is behind a corporate proxy otherwise you can skip this step.
   Goto Manage Jenkins -> Manage Plugins -> Advanced(tab) -> HTTP Proxy Configuration
   ![image](https://user-images.githubusercontent.com/14373178/152644367-ff4b7919-0716-4aad-8c40-e8fae00396df.png)
   Your server's proxy server and port details can be found from the below shell command
   ```
   $ env | grep proxy
   ```
   You can validate your proxy in the **Advanced**(tab) like this,
   ![image](https://user-images.githubusercontent.com/14373178/152644492-dd8e31c4-2f5f-4b0b-8962-3a411276fbdf.png)
   
 3. Now you can install the plugins that you want without any issues. Skip this step if you have Git plugin already installed.
    Goto Manage Jenkins - Manage Plugins -> Available(tab) -> Search for **Git** -> Select the plugin and select **Install without restart** at bottom.
    This wil install Git and all its dependent plugins. We need this Git plugin for our demo.
     
### Step 4: Create a jenkins job to clone our demo code from github and schedule polling.
Select **New Item** from the Homepage. Give the name as **Demo** and select **Freestyle project**.
Under **Source Code Mangement**, Select **Git** and Enter this repo url.
https://github.com/RajasekarVeeran/demo-cicd-jenkins-docker.git
![image](https://user-images.githubusercontent.com/14373178/152644925-2fa1339d-d438-4ea4-b206-b52c251795fb.png)

Replace the branch name from \*/master to \*/main (because that's the default branch for our demo repo)
![image](https://user-images.githubusercontent.com/14373178/152645066-cbe3a79c-b5eb-42de-89a8-00f04eba9500.png)

Under **Build Triggers** select **Poll SCM** and give 'H/5 * * * *' value for the **Schedule** field.
This will do polling for every 5 minutes and initiate a build if any changes identified in the repo.
![image](https://user-images.githubusercontent.com/14373178/152645195-48e2a6e0-b115-4feb-bc0d-56b51715ae3a.png)

Select Save. Well done! you have successfully created your first jenkins job.
Now Selecct Build Now option in your project page and check the console output to see if the build is success.
If not check the repo url for any typos.

### Step 5: Deploy our demo frontend app as docker container
Goto this path
```
$ cd ~/jenkins-data/jenkins_home/workspace/Demo/
```
and run the below command to create and run our frontend app container hosted in apache webserver.
```
$ docker-compose up -d
```
Run ```docker ps``` and check if the container is up and running. If not check the container logs for details and fix it.
You should get an output similar to this,
![image](https://user-images.githubusercontent.com/14373178/152645716-7e3addde-1297-46fd-900f-02a33ce256ba.png)

If all good, type this url in your browser and you should see a sample html page with a welcome message.
```http://<server-name>``` [OR] ```http://<server-ip>```


*That's the end of this demo. Hope it was helpful.*
