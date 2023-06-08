

## Project 9 - Tooling Website Deployment Automation With Continuous Integration

![infrastructure diagram](./images/infrastructure_diagram.png)

## Step 1 - Recreate the Architecture in the diagram from the previous labs

## Step 2 - Install a Jenkins server

> Lauch a new EC2 instance based on Ubuntu Server 20.04 LTS and name it "Jenkins"

  ![instances](./images/instances.png)

> Install JDK (since Jenkins is a Java-based application)

    sudo apt update

  ![updating repository](./images/updating_repository.png) 

    sudo apt install default-jdk-headless

  ![installing_jdk](./images/installing_Jdk.png)

> Install Jenkins
    https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
---
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
  ![jenkins](./images/jenkins.png)
---
    sudo apt update
    sudo apt-get install jenkins

  ![installing_jenkins](./images/installing_jenkins.png)

> Verify jenkins is up and running

    sudo systemctl status jenkins

![jenkins_status](./images/jenkins_status.png)

> Jenkins server uses TCP port 8080 - open it by adding a new inbound rule in your EC2 security group

  ![jenkins_port8080](./images/jenkins_port8080.png)


> Perform initial jenkins setup

  http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

 ![jenkins_initial_setup](./images/jenkins_initial_setup.png)

> To unlock jenkins, retrieve the password frm the server

    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

![initial_password](./images/initial_password.png)

c40a80b3a94c419e8cdcc4c5a130c365

> Install Jenkins plugins

![install_plugins](./images/install_jenkins_plugins.png)

![jenkins_setup](./images/jenkins_setup.png)

> Configure Jenkins to retrieve source code from GitHub using Webhooks

![github_webhooks](./images/github-webhooks.png)

> Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![new_jenkins_project](./images/new_jenkins_project.png)

> Connect to your GitHub repository in Jenkins by provding the repostory URL

![github_url](./images/github_jenkins.png)


> Save the configuration and run the build, Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![build](./images/build.png)

> Click "Configure" your job/project and add these two configurations

  - Configure triggering the job from GitHub webhook

![build_trigger](./images/build_triggers.png )

  - Post-build Actions

  Click on the drop down arrow and select "Archive the artifact" and then save.

![post_build_actions](./images/post-build-action.png)

![archive_artifact](./images/archive_artifact.png)

> Now to trigger the build, make changes to repo and commit changes to master branch.

![editing readme file](./images/updating_repository.png)

![triggered](./images/triggered.png)

> Artifacts are stored on jenkins server locally 

    ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
![Artifact stored locally on the jenkins server](./images/file_on_jenkins_server.png)

> Configure Jenkins to copy files to NFS server via SSH

![jenkins_plugin](./images/jenkins_plugin.png)

> Configure the job/project to copy artifacts over to NFS server.

- On main dashboard select "Manage Jenkins" under the "System Configuration menu" choose "System" menu item.

![manage_jenkins](./images/manage_jenkins.png)

- Scroll down to "Publish over SSH" plugin configuration section and configure it to be able to connect to your NFS server and provide the following details:

![publish_over_ssh](./images/publish_over_ssh1.png)

- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

- Arbitrary name
- Hostname – can be private IP address of your NFS server
- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
![publish_over_ssh](./images/publish_over_ssh2.png)

- Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![ssh_success](./images/ssh_success.png)

> Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"

![build_action](./images/build.png)

> Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.
If you want to apply some particular pattern to define which files to send – use this syntax.

![send_over_ssh_configuration](./images/send_over_ssh.png)

> Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

- Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:
![files_transferred_to_ssh_successfully](./images/success.png)

> To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file

    cat /mnt/apps/README.md

![ssh_success_on_nfs_server](./images/ssh_success_on_nfs_server.png)

