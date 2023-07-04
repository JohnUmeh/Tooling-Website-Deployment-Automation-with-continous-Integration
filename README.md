# Tooling-Website-Deployment-Automation-with-continous-Integration
Using Jenkins to Automate Source code Changes on our Devops Team tool.

In the [previous project 8](https://github.com/JohnUmeh/Load-Balancer-Solution-With-Apache) we introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and we successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. Manual configuration is easy with few number of servers like 2 or 3 but automation of this configuration is neccessary when then number of servers are in hundreds or larger sum. We will be using [Jenkins](https://www.jenkins.io/) to automate source code management in this project- in this project we are going to utilize Jenkins CI capabilities to make sure that every change made to the [source code in GitHub](https://github.com/JohnUmeh/Devops_Tooling_website_Solution) will be automatically be updated to the Tooling Website.

This is the current project architecture

![Screenshot from 2023-03-22 22-59-47](https://user-images.githubusercontent.com/77943759/227048296-c10fbef2-a0dd-4aa9-b200-0561ba796c2b.png)


This is how the new website Architecture would be after the automation

![Screenshot from 2023-03-22 22-57-40](https://user-images.githubusercontent.com/77943759/227048016-2acd1098-dc14-4200-a764-16f6a3b5b9ce.png)

## **INSTALL AND CONFIGURE JENKINS SERVER**

Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

![Screenshot from 2023-03-22 23-10-04](https://user-images.githubusercontent.com/77943759/227049809-4ead68fc-b345-4383-89ea-f61a83fe8468.png)

Install JDK (since Jenkins is a Java-based application)

```
sudo apt update
sudo apt install default-jdk-headless
```
Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```

Check jenkins is up and running:

`sudo systemctl status jenkins`

Jenkins server uses TCP port 8080, open the inbound rule in your ec2 instance

![Screenshot from 2023-03-22 23-16-41](https://user-images.githubusercontent.com/77943759/227050846-0933a572-dfad-437a-9f23-9b33f316e3a5.png)

Setup Jenkins

From your browser access: 
`http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`

![Screenshot from 2023-03-22 23-20-23](https://user-images.githubusercontent.com/77943759/227051445-2cc4c38a-334d-4e9c-91e5-d5c927ae56a1.png)

Get the default admin password

Retrieve it from your server:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Choose suggested plugins when prompted to choose plugin to install

![Screenshot from 2023-03-22 23-25-20](https://user-images.githubusercontent.com/77943759/227052081-392b0f78-bb04-4a25-a6c7-f95d5269e61d.png)

When the pluggins is done installing; Create an admin user and you will get your Jenkins server address

![Screenshot from 2023-03-22 23-29-27](https://user-images.githubusercontent.com/77943759/227052822-ec68eb8a-d663-4b36-b93b-562508b84b2d.png)

Installation is complete and ready for use.


## **Configure Jenkins to retrieve source codes from GitHub using Webhooks**

In this part, we will configure a simple jenkins job/project. This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Open the tooling project on git hub

2. Click on settings

![Screenshot from 2023-03-22 23-43-01](https://user-images.githubusercontent.com/77943759/227054758-ae0a53d0-b05f-4cc8-9f47-ad8bafcd042c.png)


3. Click on webhook

4. Enter `http://<jenkins-server-public-ip-address>:8080/github-webhook/`

5. Select application/json

![webhook](https://user-images.githubusercontent.com/77943759/227054485-f84aed61-2d4f-4483-b4bf-ed16f076977b.png)


Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![projectjedk](https://user-images.githubusercontent.com/77943759/227055048-ffe7cf3e-a1b0-48be-8840-58811e2cfde2.png)

To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

![Screenshot from 2023-03-22 23-47-23](https://user-images.githubusercontent.com/77943759/227055358-a9f7a52e-2dac-479a-8afb-95f313892bcc.png)


In configuration of the Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![Screenshot from 2023-03-17 02-10-14](https://user-images.githubusercontent.com/77943759/227056159-3cac9f72-f9f2-4e8d-b3a9-1a207a72b0b6.png)


![Screenshot from 2023-03-22 23-54-28](https://user-images.githubusercontent.com/77943759/227056315-699b2df2-98e5-49e7-a907-1bb0cb523a52.png)

Save the configuration and let us try to run the build

Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![build](https://user-images.githubusercontent.com/77943759/227056731-f82e6073-d6d2-44ae-9b56-9901907e007e.png)

You can open the build and check in "Console Output" if it has run successfully

Let us make this project build run automatically since it still runs manually at this stage:

Click "Configure" your job/project and add these two configurations
Configure triggering the job from GitHub webhook:

![Screenshot from 2023-03-23 00-01-04](https://user-images.githubusercontent.com/77943759/227057554-9342d972-2bf3-41ac-b6a2-d0ff25a18f7f.png)


![archive_artifacts](https://user-images.githubusercontent.com/77943759/227058027-24310aa0-02ec-4afb-b273-2f82bd1ad96e.gif)

Go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

![artifact](https://user-images.githubusercontent.com/77943759/227058247-ec5e3b5c-3a97-4b4c-a283-4d21cd5a34d1.png)

By default, the artifacts are stored on Jenkins server locally

ls /var/lib/jenkins/jobs/<Project-tooling_github-name>/builds/<build_number>/archive/


## **CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH**

Since we now have the artifacts saved locally in our Jenkins Server, we want to copy them to the NFS SERVER to the /mnt/apps directory

Install Publish Over SSH plugin

On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it

![pOSSH](https://user-images.githubusercontent.com/77943759/227060330-ad921ec3-32fd-42e2-b64a-ca174510c4f5.png)


Configure the job/project to copy artifacts over to NFS server.

On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

- A name

- Hostname – can be private IP address of your NFS server

- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)

- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

Ensure to test and get the success message before saving.

![successtest](https://user-images.githubusercontent.com/77943759/227059773-990262b2-ee0f-4f43-b062-11051d6e81fa.png)

Save the configuration

Open your Jenkins job/project configuration page and add another one "Post-build Action"

![send_build](https://user-images.githubusercontent.com/77943759/227060497-beb5865a-6dea-47d7-b6c3-c29d88d9982c.png)

Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.

![artifactssh](https://user-images.githubusercontent.com/77943759/227060685-9f6fe312-5039-46c0-be50-4dd12a861af9.png)

Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

![consoleoutput](https://user-images.githubusercontent.com/77943759/227061300-379cfe37-b4a9-474e-b6a0-131e12c25d7a.png)


Connect to your server through the terminal and run

`cat /mnt/apps/README.md`

![readmeedit](https://user-images.githubusercontent.com/77943759/227061647-7668e943-6c03-4560-9359-62333b940147.png)


If you see exactly the change you made on your READme file, the job has successfully been set up.

End




