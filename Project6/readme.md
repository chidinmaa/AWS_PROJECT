## WEB SOLUTION WITH WORDPRESS
This is a PHP-based solution.
This project prepares storage infrastructure on two Linux servers with a basic web solution using WordPress. WordPress is a free and open-
source content management system written in PHP and paired
with MySQL or MariaDB as its backend Relational Database Management System
(RDBMS).
Consisting of two parts:
1. Configure storage subsystem for Web and Database servers based on Linux OS.

2. Install WordPress and connect it to a remote MySQL database server. 
Three-tier Architecture is a client-server software architecture pattern that comprise of
3 separate layers.LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS
“WEB SERVER”.
Step 1 — Prepare a Web Server
-  Launch an EC2 instance that will serve as Web Server. Create 3 volumes in the
same AZ as your Web Server EC2, each of 10 GiB.


```
sudo gdisk /dev/xvdf
```
- Use this to partition.
```
sudo fdisk /dev/xvdf
sudo fdisk/dev/xvdg
sudo fdisk/dev/xvdh
```
- Use lsblk utility to view the newly configured partition on each of the 3 disks.
```
sudo lsblk
```
- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
```
sudo yum install lvm2
sudo lvmdiskscan
```

Note: Unlike ubuntu that uses apt, for redhat the package manager is yum.

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```
sudo pvcreate /dev/xvdf
sudo pvcreate /dev/xvdg
sudo pvcreate /dev/xvdh
```
- Verify that your Physical volume has been created successfully by running sudo pvs
```
sudo pvs
```
- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```
sudo vgcreate webdata-vg /dev/xvdf2 /dev/xvdg2 /dev/xvdh2
```
- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
- Verify that your Logical Volume has been created successfully by running sudo lvs
```
sudo lvs
```
- Verify the entire setup
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```
sudo lsblk 
```
- Use mkfs.ext4 to format the logical volumes with ext4 filesystem
```
sudo mkfs.ext4 /dev/webdata-vg/apps-lv
sudo mkfs.ext4 /dev/webdata-vg/logs-lv
```
## Creating a directory structure.
Create /var/www/html directory to store website files
```
sudo mkdir -p /var/www/html
```
- Create /home/recovery/logs to store backup of log data
```
sudo mkdir -p /home/recovery/logs
```
- Mount /var/www/html on apps-lv logical volume
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step of creating /var/www/html directory to store website files)
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```
- Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/. /var/log
```
## UPDATING THE /ETC/FSTAB FILE
Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file;
```
sudo blkid
```
Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
```
sudo nano /etc/fstab
```
and add this
```
UUID=<uuid of your webdata-vg-apps> /var/www/html ext4 defaults 0 0
UUID=<uuid of your webdata-vg-logs> /var/log ext4 defaults 0 0
```
- Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```
- Verify your setup by running df -h, output must look like this:
```
sudo df -h
```
## Preparing the Database Server
- Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

- Create and attach 3 Logical Volumes to the database server instance.
- Connect to your linux server and check if the volume is attached using this command:
```
sudo lsblk
```
- Use df -h command to see all mounts and free space on your server
```
sudo df -h
```
Using gdisk utility create a single partition on each of the 3 disks
sudo fdisk /dev/xvdf
and then create a new partition with the "n" command, inputting 1 (to create a single partition) and then the "w" command and enter "y" to create a single partition.

Also repeat the same for the other two disks
```
sudo fdisk /dev/xvdg
sudo fdisk /dev/xvdh
```
- Use lsblk utility to view the newly configured partition on each of the 3 disks.
```
sudo lsblk
```
- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
```
sudo yum install lvm2
sudo lvmdiskscan
```
Note: Unlike ubuntu that uses apt, for redhat the package manager is yum.

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```
sudo pvcreate /dev/xvdf2 /dev/xvdg2 /dev/xvdh2
```
- Verify that your Physical volume has been created successfully by running sudo pvs
```
sudo pvs
```
- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```
sudo vgcreate dbdata-vg /dev/xvdf2 /dev/xvdg2 /dev/xvdh2
```
- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
```
sudo lvcreate -n db-lv -L 14G dbdata-vg
sudo lvcreate -n logs-lv -L 14G dbdata-vg
```
Verify that your Logical Volume has been created successfully by running sudo lvs
```
sudo lvs
```
- We need to verify all we have done so far on the database server instance so far with these commands.
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```
sudo lsblk
```

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem
```
sudo mkfs.ext4 /dev/dbdata-vg/db-lv && sudo mkfs.ext4 /dev/dbdata-vg/logs-lv
```
- Now that we are done configuring the database logical volumes, we would be moving on with creating the mount points for the logical volumes and the required directories.

- Create /db directory to store website files
```
sudo mkdir -p /db
```
- Create /home/recovery/logs to store backup of log data
```
sudo mkdir -p /home/recovery/logs
```
- Mount /db on apps-lv logical volume
```
sudo mount /dev/dbdata-vg/db-lv /db
``` 
- Like we did for the webserver instance use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step of creating /db directory to store database files)
```
sudo mount /dev/dbdata-vg/logs-lv /var/log
```
- Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/. /var/log
```
Now we need to update the /etc/fstab file to ensure that the configurations we made is persistent across reboots.

Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file;
```
sudo blkid
```
- Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
```
sudo nano /etc/fstab
```
and add this
```
UUID=<uuid of your webdata-vg-apps> /var/www/html ext4 defaults 0 0
UUID=<uuid of your webdata-vg-logs> /var/log ext4 defaults 0 0
```
- Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```
Verify your setup by running df -h, output must look like this:
```
sudo df -h
```
Now your db server is ready to go and make other configurations as required.

## Install WordPress on your Web Server EC2 Instance
- Update the repository
```
sudo yum update
```
- Install wget, Apache and it’s dependencies
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
- Start the apache service
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
To install PHP and it’s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```
## Restart Apache
```
sudo systemctl restart httpd
```
- Download wordpress and copy wordpress to var/www/html
```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/
```
- Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```
## Install MySQL on your DB Server EC2

- Install mysql on the db-server
```
sudo yum update
sudo yum install mysql-server
```
- We now need to verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
## Configuring the DB to work with WordPress
Here we need to configure the database to work with WordPress. By allowing the wordpress server be able to connect to the database, we need to configure the database to allow the wordpress server to connect to the database.
We need to then create a user for the wordpress server to connect to the database.
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
## Configure WordPress to connect to remote database In My WebServer
- Here we are to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32.

- Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
```
sudo yum install mysql
```
Create login to the database on the db server
```
sudo mysql -u <user> -p -h <DB-Server-Private-IP-address>
```
- Note the is the name of the user you created in mysql server on the db server.
Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.
```
SHOW DATABASES;
```
![alt text](<Images/show databases.PNG>)


Change permissions and configuration so Apache could use WordPress:
Here we need to create a configuration file for wordpress in order to point client requests to the wordpress directory. Open port 80 in your webserver.
```
sudo vi /etc/httpd/conf.d/wordpress.conf
```
- and copy and paste the lines below:
```
<VirtualHost *:80>
ServerAdmin myuser@3.88.215.221
DocumentRoot /var/www/html/wordpress

<Directory "/var/www/html/wordpress">
Options Indexes FollowSymLinks
AllowOverride all
Require all granted
</Directory>

ErrorLog /var/log/httpd/wordpress_error.log
CustomLog /var/log/httpd/wordpress_access.log common
</VirtualHost>
```
To apply the changes, restart Apache
```
sudo systemctl restart httpd
```
Edit the wp-config file
```
sudo vi /var/www/html/wordpress/wp-config.php
```
and add the following lines:
```
define('DB_NAME', 'wordpress');
define('DB_USER', 'myuser');
define('DB_PASSWORD', 'mypass');
define('DB_HOST', '<db-Server-Private-IP-Address>');
define('DB_CHARSET', 'utf8mb4');
define('DB_COLLATE', '');
```
- configure SELinux for wordpress
```
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/wordpress/.*?"
```
- Note: The semanage command is not available on CentOS 7.x.x. and you might need to install it using the following command:
```
sudo yum provides /usr/sbin/semanage
sudo yum install policycoreutils-python-utils
```
- Try to access from your browser the link to your WordPress
```
http://<Web-Server-Public-IP-Address>/
```

![alt text](<Images/wordpress dashboard.PNG>)

![alt text](Images/wordpress-2.PNG)

![alt text](Images/wordpress2-1.PNG)




