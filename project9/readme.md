 TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. IN
TRODUCTION TO JENKINS
In previous Project 8, we introduced the horizontal scalability concept, which allows
us to add new Web Servers to our Tooling Website. You have successfully deployed a
set-up with 2 Web Servers and a Load Balancer to distribute traffic between them. If it is
just two or three servers – it is not a big deal to configure them manually. Imagine that
you would need to repeat the same task repeatedly adding dozens or even hundreds of
servers.
DevOps is about Agility and the speedy release of software and web solutions. One of
the ways to guarantee fast and repeatable deployments is the Automation of routine
tasks.
In this project, we are going to start automating part of our routine tasks with a free and
open-source automation server – Jenkins. It is one of the most popular CI/CD tools, it
was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the
project originally had a named &quot;Hudson&quot;.
According to Circle CI, Continuous integration (CI) is a software development strategy
that increases the speed of development while ensuring the quality of the code that
teams deploy. Developers continually commit code in small increments (at least daily, or
even several times a day), which is then automatically built and tested before it is
merged with the shared repository.
In our project we are going to utilize Jenkins CI capabilities to make sure that every
change made to the source code in GitHub https://github.com/&lt;yourname&gt;/tooling will
be automatically be updated to the Tooling Website.
### INSTALL AND CONFIGURE JENKINS SERVER
####  Step 1 – Install the Jenkins server
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it
&quot;Jenkins&quot;
2. Install JDK (since Jenkins is a Java-based application)
```
sudo apt update
sudo apt install default-jdk-headless
```
1. Install Jenkins
```
sudo apt install openjdk-8-jdk
```
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
e>     /etc/apt/sources.list.d/jenkins.list'
```
```
sudo apt-get update
sudo apt-get install jenkins -y
```
- Make sure Jenkins is up and running
```
sudo systemctl status jenkins
```
- By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your Ec2 security group.

![alt text](<jenkins inbound rule.PNG>)

- Perform initial Jenkins setup.
From your browser access http://Jenkins-Server-Public-IP-Address-or-Public-DNSName:8080
You will be prompted to provide a default admin password.
![alt text](<unlock jenkins.PNG>)

Retrieve it from your server:
```
sudo cat /var/lib/jenkins/secrets/
```
initialAdminPassword
Then you will be asked which plugings to install – choose suggested plugins.
![alt text](<customized jenkins.PNG>)
Once plugin installation is done – create an admin user and you will get your Jenkins
server address.
The installation is completed!
- Use admin,admin,admin@admin.com
![alt text](<jenkins is ready.PNG>)

Step 2 – Configure Jenkins to retrieve source codes from GitHub
using Webhooks
In this part, you will learn how to configure a simple Jenkins job/project (these two terms
can be used interchangeably). This job will be triggered by GitHub  webhooks  and will
execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins
server.
- Enable webhooks in your GitHub repository settings
```
http://jenkins.example.com/github-webhook/
```
- Enable webhooks in your GitHub repository settings. NOTE: end the payload url with /github-webhook/ 

- Go to Jenkins web console, click "New Item" and create a "Freestyle project" a

- To connect your GitHub repository, you will need to provide its URL, you can copy it
from the repository itself.
- In the configuration of your Jenkins freestyle project choose Git repository, and provide
there the link to your Tooling GitHub repository and credentials (user/password) so
Jenkins could access files in the repository.
- Dont forget the master to main.
- Save the configuration and let us try to run the build. For now, we can only do it
manually.
Click the Build Now button, if you have configured everything correctly, the build will
be successful and you will see it.
- You can open the build and check in Console Output if it has run successfully.
If so – congratulations! You have just made your very first Jenkins build!
But this build does not produce anything and it runs only when we trigger it manually.
Let us fix it.
- Click &Configure your job/project and add these two configurations
Configure triggering the job from the GitHub webhook:
- Configure Post-build Actions to archive all the files – files resulting from a build are
called &artifacts.

- Now, go ahead and make some changes in any file in your GitHub repository
(e.g. README.MD file) and push the changes to the master branch.
You will see that a new build has been launched automatically (by webhook) and you
can see its results – artifacts, saved on the Jenkins server.

- You have now configured an automated Jenkins job that receives files from GitHub by
webhook trigger (this method is considered as ‘push’ because the changes are being
‘pushed’ and file transfer is initiated by GitHub). There are also other methods: trigger
one job (downstream) from another (upstream), poll GitHub periodically and others.
By default, the artifacts are stored on the Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/

ls /var/lib/jenkins/jobs/tooling_github/builds/3
```

### CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH
Step 3 – Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them
to our NFS server to /mnt/apps directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We
will need a plugin that is called Publish Over SSH.
- Install the Publish Over SSH plugin.
On the main dashboard select Manage Jenkins and choose the Manage Plugins
menu item.
On the Available tab search for the Publish Over SSH plugin  and install it.

Configure the job/project to copy artifacts over to the NFS server.
On the main dashboard select Manage Jenkins and choose the Configure System
menu item.
Scroll down to Publish over the SSH plugin configuration section and configure it to be
able to connect to your NFS server:
- Provide a private key the content of .pem file that you use to connect to the NFS
server ssh key pem.
-  Arbitrary name
-  Hostname – can be private IP address of your NFS server
- Username – ec2-user (since the NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mounting point
to retrieve files from the NFS server.
- Test the configuration and make sure the connection returns Success. Remember, that
TCP port 22 on NFS server must be open to receive SSH connections.

Save the configuration, open your Jenkins job/project configuration page and add
another one Post-build Action.
![alt text](Build.PNG)

- Configure it to send all files produced by the build into our previously define remote
directory. In our case we want to copy all files and directories – so we use **.
If you want to apply some particular pattern to define which files to send – use this
syntax.
![alt text](<ssh build.PNG>)

- Save this configuration and go ahead, and change something in README.MD file in your
GitHub Tooling repository.
Webhook will trigger a new job and in the Console Output of the job you will find
something like this:
- you could encounter some problems.
go to your NFS server terminal and type
```
cd /mnt/apps
sudo chmod 777 /mnt/apps
sudo chown nobody:nobody /mnt/apps

sudo chmod -R 777 /mnt/apps
sudo chown -R nobody:nobody /mnt/apps
```

SSH: Transferred 25 file(s)
Finished: SUCCESS
To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to
your NFS server and check README.MD file
cat /mnt/apps/README.md
If you see the changes you had previously made in your GitHub – the job works as
expected.
Congratulations!
You have just implemented your first Continous Integration solution using Jenkins CI.
Watch out for advanced CI configurations in upcoming projects.