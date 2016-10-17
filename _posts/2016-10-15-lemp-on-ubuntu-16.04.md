---
published: true
layout: post
comments: true
title:  "Install LEMP on Ubuntu 16.04"
date:   2016-10-15
categories:
  - Webserver
  - Nginx
---

Update Ubuntu Repository & Upgrade Packages

    sudo apt-get update
    sudo apt-get upgrade -y

Install Nginx

    sudo service nginx start
 
Install MySQL

    sudo apt-get install mysql-server
 
To secure the installation, we can run a simple security script that will ask whether we want to modify some insecure defaults. Begin the script by typing:

    sudo mysql_secure_installation
 
Install PHP
 
    sudo apt-get install php-fpm php-mysql
  
Configure the PHP
  
    vi /etc/php/7.0/fpm/php.ini
   
Uncomment the line and setting it to "0" like this: 

    cgi.fix_pathinfo=0 
 
Restart our PHP

    sudo systemctl restart php7.0-fpm

Configure Nginx to Use the PHP

    sudo vi /etc/nginx/sites-available/default

Make change as below    
```nginx
 server {
     listen 80 default_server;
     listen [::]:80 default_server;
 
     root /var/www/html;
     index index.php index.html index.htm index.nginx-debian.html;
 
     server_name server_domain_or_IP;
 
     location / {
         try_files $uri $uri/ =404;
     }
 
     location ~ \.php$ {
         include snippets/fastcgi-php.conf;
         fastcgi_pass unix:/run/php/php7.0-fpm.sock;
     }
 
     location ~ /\.ht {
         deny all;
     }
 }
```

Test your configuration file for syntax errors by typing:

    sudo nginx -t
 
 Create a PHP File to Test Configuration
 
    sudo vi /var/www/html/info.php
  
 Paste the following lines into the new file

```php 
<?php
phpinfo();
?>
```
 
Save and close the file.

Go to the URL below to test PHP

    http://server_domain_or_IP/info.php
 
 
Install PhpMyAdmin

    sudo apt-get install phpmyadmin php-mbstring php-gettext
 
Create a symbolic link from the installation files to our Nginx document root directory by typing this:
 
    sudo ln -s /usr/share/phpmyadmin /var/www/pma.micinthe.com
 
Create PhpMyadmin Server Block 
 
    sudo vi /etc/nginx/sites-available/pma.micinthe.com
 
Paste:

```nginx
server {
        listen 80;
        server_name pma.micinthe.com;
        root /var/www/pma.micinthe.com;
 
        index index.php index.html;
        location = /favicon.ico {
                 log_not_found off;
                 access_log off;
        }
        location = /robots.txt {
                 allow all;
                 log_not_found off;
                 access_log off;
        }
        # Make sure files with the following extensions do not get loaded by nginx because nginx would display the source code, and these files can contain PASSWORDS!
         location ~* \.(engine|inc|info|install|make|module|profile|test|po|sh|.*sql|theme|tpl(\.php)?|xtmpl)$|^(\..*|Entries.*|Repository|Root|Tag|Template)$|\.php_ {
                 deny all;
         }
        # Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store (Mac).
        location ~ /\. {
                 deny all;
                 access_log off;
                 log_not_found off;
        }
        location ~*  \.(jpg|jpeg|png|gif|css|js|ico)$ {
                 expires max;
                 log_not_found off;
        }
        location ~ \.php$ {
                 try_files $uri =404;
                 include /etc/nginx/fastcgi_params;
                 fastcgi_pass 127.0.0.1:9000;
                 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
}
```

Save and close the file.
 
Create a symbolic link between the sites-available directory and the sites-enabled directory. 

    sudo ln -s /etc/nginx/sites-available/pma.micinthe.com /etc/nginx/sites-enabled/pma.micinthe.com
 
Delete the default nginx server block:

    sudo rm /etc/nginx/sites-enabled/default

Install Let's Encrypt

    mkdir /opt/certbot
    cd /opt/certbot
    wget https://dl.eff.org/certbot-auto
    chmod a+x ./certbot-auto

Run cerbot-auto to generate certificate

    ./certbot-auto certonly

Example of adding certbot certificate to nginx server block:

Make the following changes to the server block:

    sudo vi /etc/nginx/sites-available/pma.micinthe.com

```nginx
 server {
        listen         80;
        server_name    pma.micinthe.com;
        return         301 https://$server_name$request_uri;
 }
 
 server {
        listen 443 ssl http2;
        server_name pma.micinthe.com;
        # add Strict-Transport-Security to prevent man in the middle attacks
        add_header Strict-Transport-Security "max-age=31536000";
        root /var/www/pma.micinthe.com;
 
        ssl_certificate /etc/letsencrypt/live/pma.micinthe.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/pma.micinthe.com/privkey.pem;
```
