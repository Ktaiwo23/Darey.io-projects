# TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

In previous Project 8 we introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and you have successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. If it is just two or three servers – it is not a big deal to configure them manually. Imagine that you would need to repeat the same task over and over again adding dozens or even hundreds of servers.

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server – Jenkins. It is one of the most popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named “Hudson”.

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub `https://github.com/<yourname>/tooling` will be automatically be updated to the Tooling Website.

### Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how our updated architecture will look like upon competition of this project:

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/225da0f8-b669-4e45-97d7-7224c43e1fcf)

## INSTALL AND CONFIGURE JENKINS SERVER

### Step 1 – Install Jenkins server
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/3aab12d8-bddc-4659-a912-60a718e3124c)

2. Install JDK (since Jenkins is a Java-based application)
   
        sudo apt update -y
        sudo apt install default-jdk-headless
   
3. Install Jenkins
   
        wget -q -O - https://pkg.jenkins.io/debian-stable/jenins.io.key |sudo gpg --dearmor -o /usr/share/keyrings/jenkins.gpg
        sudo sh -c 'echo deb [signed-by=/usr/share/keyrings/jenkins.gpg] http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
        curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
          /usr/share/keyrings/jenkins-keyring.asc > /dev/null
        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
          https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
          /etc/apt/sources.list.d/jenkins.list > /dev/null
        sudo apt update -y
        sudo apt-get install openjdk-11-jdk -y
        sudo apt install jenkins -y
   
Make sure Jenkins is up and running

`sudo systemctl status jenkins`
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/90bebc66-3810-40a6-bf14-faa4f815f5d2)

4. By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/d65ffdee-85b0-46fa-b6b2-cc5770529f36)

5. Perform initial Jenkins setup.
From your browser access `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`

You will be prompted to provide a default admin password

Retrieve it from your server:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Then you will be asked which plugins to install – choose suggested plugins.

Once plugins installation is done – create an admin user and you will get your Jenkins server address.

The installation is completed!
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/d506ce79-0579-4c8a-bc5f-e0ce4dffed89)

### Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/27460357-6f55-4589-ad7f-57544f88251e)

2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”

To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

Save the configuration and let us try to run the build. For now we can only do it manually.
Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under #1

You can open the build and check in “Console Output” if it has run successfully.
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/81e04040-4972-43c6-b110-175240da1d74)

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3. Click “Configure” your job/project and add these two configurations
Configure triggering the job from GitHub webhook:

Configure “Post-build Actions” to archive all the files – files resulted from a build are called “artifacts”.

Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/149478c8-bd52-4eca-94ff-d60e1b717dc5)

You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/7b77bed7-65b1-40ec-a2d6-e513a9fd16f8)

### Step 3 – Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to `/mnt/apps` directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called “Publish Over SSH”.

1. Install “Publish Over SSH” plugin.
On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.

On “Available” tab search for “Publish Over SSH” plugin and install it 
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/08178e67-7255-4ab7-b06a-571e06f7e6ab)

2. Configure the job/project to copy artifacts over to NFS server.
On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

* Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
* Arbitrary name
* Hostname – can be private IP address of your NFS server
* Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
* Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/4c3a324d-83cc-4c52-985f-1e0be31ede61)

Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”

Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use **.

If you want to apply some particular pattern to define which files to send – use this syntax.

Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the “Console Output” of the job you will find something like this:

        SSH: Transferred 25 file(s)
        Finished: SUCCESS
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/4f935d45-8458-4faf-9f64-0e1dd864d9ce)

To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file

        cat /mnt/apps/README.md
        
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/446b7861-41b8-4449-9dba-158c1d74b30c)
        
If you see the changes you had previously made in your GitHub – the job works as expected.

















