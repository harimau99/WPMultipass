# Prerequisite:

Prior to beginning the training or following the tutorial. To have a similar experience, it is recommended to follow 1 to 1 as outlined in the prep list below.

### Tools involve that easier to access
##
- **File Transfer**  
	- **CyberDuck** : https://cyberduck.io/download/
##
- **Database Management Tools**
	- **DB Beaver** : https://dbeaver.io/download/
##
- **Code Editor / IDE**
	- **VSCODE** : https://code.visualstudio.com/download
##
- **Environment Setup**
	- **Multipass** : https://multipass.run/
##
##
##
Once your prep is completed. You may started the step by step tutorial.

Have fun!
##
# Pre-setup
##
Download the  Multipass and install the application.

## Windows

### Install Multipass on Windows

https://multipass.run/download/windows

Note: You need Windows 10 Pro/Enterprise/Education v 1803 or later, or any Windows 10 with VirtualBox

Make sure your network is private or Windows prevents Multipass from starting.

Run the installer. You need to allow the installer to gain Administrator privileges.

How to launch LTS instances

The first five minutes with Multipass let you know how easy it is to have a lightweight cloud handy. Let’s launch a few LTS instances, list them, exec a command, use cloud-init and clean up old instances to start.

## Linux
```
$ sudo snap install multipass
```


## MacOS

Download Multipass for MacOS
https://multipass.run/download/macos

Run the installer in an account with Administrator privileges. 
If you'd like to use Multipass with VirtualBox use this terminal command:
```
$ sudo multipass set local.driver=virtualbox
```


#### Launch an instance (by default you get the current Ubuntu LTS) , in this case we setup under wptraining

```
$ multipass launch --name=wptraining
```

#### See your instances
```
$ multipass list
```

#### Stop and start instances
```
$ multipass stop wptraining

$ multipass start wptraining
```

#### Clean up what you don’t need
```
$ multipass delete wptraining

$ multipass purge
```

## Getting Started the installation
##
## Step 1: Update all dependency 

sudo apt-get update && sudo apt-get upgrade -y

## Step 2: Installation Webserver Engine, DB & related

## 2.1 - Webserver - Nginx

```
ubuntu@wptraining:~$ sudo apt-get install -y git nginx mlocate
ubuntu@wptraining:~$ sudo systemctl status nginx
ubuntu@wptraining:~$ sudo systemctl enable nginx
ubuntu@wptraining:~$ sudo systemctl restart nginx
ubuntu@wptraining:~$ sudo systemctl status nginx
```

After enable the nginx  symlink
![Screenshot 2021-09-16 at 5.07.55 AM.png](:/38828da3a2124fe4b75b4ef546a1c299)

Checking the status
![Screenshot 2021-09-16 at 5.08.30 AM.png](:/136871bb22054bd99c5f4a7709c09952)


### Lets check with this installation

Open up another tab of CLI / Command Prompt / Powershell by typing multipass command.

```
$ multipass list
```

It will show the value parameter as in the photo below

![Screenshot 2021-09-16 at 4.44.46 AM.png](:/3ea9d94d60ab4b52969efdda69f8aae8)

Look for the created Pass created recently. Copy the IP address located in IPv4 column and paste to browser.

It will look like below.

![Screenshot 2021-09-16 at 4.49.10 AM.png](:/2f12ad21276d4273a2e0c430a405f4b9)

## 2.2 - Database (DB) with MariaDB

sudo usermod -aG sudo dbauser

```
ubuntu@wptraining:~$ sudo apt-get install -y mariadb-server mariadb-client
ubuntu@wptraining:~$ sudo systemctl enable mariadb
ubuntu@wptraining:~$ sudo systemctl restart mariadb
```

After that, run the commands below to secure MariaDB server.

```
ubuntu-wptraining:~$ sudo mysql_secure_installation

```

In this sample we use this password to simulate.

##### root password : kucingC@7
##
When prompted, answer the questions below by following the guide.
```
* Enter current password for root (enter for none): Just press the Enter
* Set root password? [Y/n]: Y
* New password: Enter password
* Re-enter new password: Repeat password
* Remove anonymous users? [Y/n]: Y
* Disallow root login remotely? [Y/n]: Y
* Remove test database and access to it? [Y/n]:  Y
* Reload privilege tables now? [Y/n]:  Y
Restart MariaDB server
```

Once completed Restart mariaDB again. This just to ensure the reloaded privilege takes effect on the setup.

```
ubuntu-wptraining:~$ sudo systemctl restart mariadb
```


### Create Database User
```
create user 'wptraining'@'localhost' identified by 'kucingC@7';
```
### Create Database Schema
```
create database wpDB;
```
### Grant all privileges on created Database Schema
```
grant all privileges on wpDB.* to 'wptraining'@'localhost' with grant option;
```
##
### Troubleshooting
If you have any issues in connecting via mysql workbench or any other RDBMS tools. Kindly rerun with this settings.

```
* Enter current password for root (enter for none): Just press the Enter
* Change the root password? [Y/n]: n
* Remove anonymous users? [Y/n]: n
* Disallow root login remotely? [Y/n]: n
* Remove test database and access to it? [Y/n]:  n
* Reload privilege tables now? [Y/n]:  Y
Restart MariaDB server
```

![Screenshot 2021-09-16 at 9.42.02 AM.png](:/af2c6fe8ffb140209366c566aec12c51)


## PHP and its libraries
```
ubuntu-wptraining:~$ sudo apt-get install -y php php-fpm php-mysql php-common php-cli php-json php-opcache php-readline php-mbstring php-xml php-gd php-curl php-zip php-tcpdf php-twig
```

# Nginx
## Configure the Nginx
```
ubuntu@wptraining:~$ sudo rm /etc/nginx/sites-enabled/default
ubuntu@wptraining:~$ sudo nano /etc/nginx/conf.d/default.conf
```

#put this on the default.conf
#####################

server {
  listen 80;
  listen [::]:80;
  server_name _;
  root /usr/share/nginx/html/;
  index index.php index.html index.htm index.nginx-debian.html;

  location / {
    try_files $uri $uri/ /index.php;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php7.2-fpm.sock;   # kindly update according the access socket created
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
  }


 ## A long browser cache lifetime can speed up repeat visits to your page
  location ~* \.(jpg|jpeg|gif|png|webp|svg|woff|woff2|ttf|css|js|ico|xml)$ {
       access_log        off;
       log_not_found     off;
       expires           360d;
  }

  ## disable access to hidden files
  location ~ /\.ht {
      access_log off;
      log_not_found off;
      deny all;
  }
}


####################

To test nginx configuration:

sudo nginx -t 

If the test is successful, reload the nginx services:

sudo systemctl reload nginx


####################

If it you are having issue with error 502. Thats mean you didn't update your access socket for PHP-FPM in /etc/nginx/sites-available/default .

Step 1:  locate the information using function locate.
```
ubuntu@wptraining:~$ locate fpm
```
Look for this particular path. For this example, the php version is using php v7.4. 
```
/etc/php/7.4/fpm/pool.d/www.conf
```
![Screenshot 2021-09-24 at 9.49.56 PM.png](:/93ebb43924aa4313bcc4e1324b76c9c9)

Once you able to locate the path. Lets open the file using editor.
```
ubuntu@wptraining:~$  sudo nano /etc/php/7.4/fpm/pool.d/www.conf
```
Look for the parameter that mentioned 
```
listen = /run/php/7.x-fpm.sock
```

Press CTR+W
![Screenshot 2021-09-24 at 10.07.30 PM.png](:/16c1d2168cc943dd814575993e1b3b41)
Once you saw this search function appear. Key in the phrase  
```
listen = /run/php
```
![Screenshot 2021-09-24 at 10.07.18 PM.png](:/2ad557de3de94c98905203c2df7c37cf)

It the parameter is matched it will highlight at the beginning of the paragraph / line.

![Screenshot 2021-09-24 at 10.13.50 PM.png](:/0c2ce2e513e84c4b934e8b97c1ccf8d2)

Copy that particular information 

![Screenshot 2021-09-24 at 10.15.33 PM.png](:/d5b3fc1eb54b4c1ca11bc2bed13899fc)

and paste in the default.conf earlier that we made in this directory.
/etc/nginx/conf.d
```
ubuntu@wptraining:~$ sudo nano /etc/nginx/conf.d/default.conf
```

location ~ \.php$ {
    fastcgi_pass unix:/run/php/php7.4-fpm.sock;   # kindly update according the access socket created
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
  }


## The End





