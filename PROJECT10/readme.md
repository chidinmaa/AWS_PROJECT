### LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS
# Task
This project consists of two parts:
- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates.
- Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
```
 sudo nano /etc/hosts
 ```
 ![alt text](Nginx.Ip.JPG)
 ```
 sudo apt update
 sudo apt install nginx
 ```
 ```
 sudo vi /etc/nginx/nginx.conf
 ```
 ```
 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }
  ```
 -  comment out this line #include /etc/nginx/sites-enabled/*;
 ![alt text](upstream.JPG)
 ```
 sudo systemctl restart nginx sudo systemctl status nginx
 ```
 - REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
 - Create a hosted zone
 - I used namescheap to host a DNS name. (paschaline.com)
 ![alt text](hostzone.JPG)
 ![alt text](custom.Dns.JPG)
 - Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>
 ```
 http://paschaline.com
 ```
 - Install certbot and request for an SSL/TLS certificate Make sure snapd service is active and running
```
sudo systemctl status snapd Install certbot sudo snap install --classic certbot
```
- Request your certificate
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
sudo certbot --nginx
```
![alt text](Login.PNG)