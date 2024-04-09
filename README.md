# Installation of lampstack on AWS
## Setting up an instance
* Connected to two Ec2 instances on Aws one for mobaxtem and the other for putty and created a t3micro ubuntu server.
![alt text](<Images/aws capture.PNG>)
* Attached a .pem keypair and launched the instance and saved the key in my file explorer. 

* Used both putty and Mobaxterm terminal connecting through ssh.16.170.206.49 and 16.171.138.84.

- Using putty terminal convert the .pem key to .ppk using puttygen.Do not convert the key in mobaxterm.Make sure to connect to the region closest to you.
### Install Apache
- Install Apache and expose it to port 80 through edit inbound rule.
```
# update a list of packages in package manager
$ sudo apt update
#run apache2 package installation
$ sudo apt install apache2

$ sudo systemctl status apache2
```
[alt text](<Images/index capture.PNG>)
### EDIT INBOUND RULE IN AWS
Make sure to open port 80 in aws. This will enable us to access it from the internet locally.

```
 http://<Public-IP-Address>:80
```
### INSTALL MYSQL
Create a virtual host for your website using Apache.
```
$ sudo apt install mysql-server

$ sudo mysql_secure_installation

$ sudo mysql
```
To exit the MySQL console, type:
mysql> exit
- Enable php on the website. 
### INSTALLING PHP
```
$ sudo apt install php libapache2-mod-php php-mysql

$ sudo mkdir /var/www/projectlamp

$ sudo vi /etc/apache2/sites-available/projectlamp.conf

PASTE THIS INTO THE VI
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Then save with WQ.
![alt text](<Images/php capture.PNG>)

!


Now go to your browser and try to open your website URL using IP address:
http://<Public-IP-Address>:80
![alt text](<Images/ubuntu capture.PNG>)