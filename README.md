# Installation of lampstack on AWS
## Setting up an instance
* Connected to an Ec2 instance on Aws and created a t3micro ubuntu server.
![alt text](<aws capture.PNG>)
* Attached a .pem keypair and launched the instance and saved the key in my file exlorer. 

* Used both putty and Mobaxterm terminal connecting through ssh.16.170.206.49 and 16.171.138.84.

- Using putty terminal convert the .pem key to .ppk using puttygen.
### Install Apache
- Install Apache and expose it to port 80 through edit inbound rule.
```
sudo apt update
```

- Insatll mysql and php
Create a virtual host for your website using Apache.
```

```

- Enable php on the website. The pictures are available.