We introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and  we have successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. If it is just two or three servers – it is not a big deal to configure them manually. Imagine that  we would need to repeat the same task over and over again adding dozens or even hundreds of servers.

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

Now we are going to start automating part of our routine tasks with a free and open source automation server – Jenkins. It is one of the most popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com//tooling will be automatically be updated to the Tooling Website.

## Install Jenkins server

1, Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

2, Install JDK (since Jenkins is a Java-based application)
```
sudo apt update
sudo apt install default-jdk-headless
```
3, Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
Make sure Jenkins is up and running
```
sudo systemctl status jenkins
```
4, By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in   EC2 Security Group
5, Perform initial Jenkins setup. From   browser access http://:8080
 we will be prompted to provide a default admin password
Retrieve it from   server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Then  we will be asked which plugings to install – choose suggested plugins.

Once plugins installation is done – create an admin user and  we will get   Jenkins server address.

The installation is completed!

## Configure Jenkins with Github
Configure Jenkins to retrieve source codes from GitHub using Webhooks In this part,  we will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1, Enable webhooks in   GitHub repository settings
2, Go to Jenkins web console, click "New Item" and create a "Freestyle project"
- To connect   GitHub repository,  we will need to provide its URL,  we can copy from the repository itself
- In configuration of   Jenkins freestyle project choose Git repository, provide there the link to   Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
- Save the configuration and let us try to run the build. For now we can only do it manually. Click "Build Now" button, if  we have configured everything correctly, the build will be successfull and  we will see it under #1
  
 we can open the build and check in "Console Output" if it has run successfully.

If so   we have just made   very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3, Click "Configure"   job/project and add these two configurations Configure triggering the job from GitHub webhook:
Now, go ahead and make some change in any file in   GitHub repository (e.g. README.MD file) and push the changes to the master branch.

 we will see that a new build has been launched automatically (by webhook) and  we can see its results – artifacts, saved on Jenkins server.

 we have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
## Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

1, Install "Publish Over SSH" plugin. On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.
On "Available" tab search for "Publish Over SSH" plugin and install it
2, Configure the job/project to copy artifacts over to NFS server. On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to  NFS server:
- Provide a private key (content of .pem file that  we use to connect to NFS server via SSH/Putty)
- Arbitrary name
- Hostname – can be private IP address of  NFS server
- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

Save the configuration, open  Jenkins job/project configuration page and add another one "Post-build Action"

Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.

If  we want to apply some particular pattern to define which files to send – use this syntax.

Save this configuration and go ahead, change something in README.MD file in  GitHub Tooling repository.

Webhook will trigger a new job and in the "Console Output" of the job  we will find something like this:
```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```
To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to  NFS server and check README.MD file
```
cat /mnt/apps/README.md
```
If  we see the changes  we had previously made in  GitHub – the job works as expected.

Congratulations! we have just implemented our first Continous Integration solution using Jenkins CI.
