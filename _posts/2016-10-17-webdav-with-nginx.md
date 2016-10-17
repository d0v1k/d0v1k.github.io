---
published: true
layout: post
comments: true
title:  "WebDAV with nginx"
date:   2016-10-17
categories:
  - Howtos
  - Web Server
---

### Install Let's Encrypt

    mkdir /opt/certbot
    cd /opt/certbot
    wget https://dl.eff.org/certbot-auto
    chmod a+x ./certbot-auto

### Certificate request

Stop the nginx daemon

    systemctl stop nginx

Run cerbot-auto to generate certificate

    ./certbot-auto certonly

    _Select 2 Spin up a temporary webserver (standalone)
    Enter you email address
    Enter Domain name_

### Create webdav virtualhost

    mkdir -p /var/www/webdav.example.com
    sudo chown www-data.www-data webdav.example.com

Generate a dhparam.pem file :

    mkdir /etc/nginx/ssl && chmod 700 /etc/nginx/ssl
    openssl dhparam -out /etc/nginx/ssl/dhparam.pem 3072
    chmod 600 /etc/nginx/ssl/dhparam.pem

Let’s now generate a HTTP basic authentication file. This example creates a user named example :

    mkdir /etc/nginx/auth
    htpasswd -c /etc/nginx/auth/webdav example
    New password: 
    Re-type new password: 
    Adding password for user user

This file has to be readable by the user running your webserver. For security reasons, we’ll make it readable only by him :

    chown -R www-data:nogroup /etc/nginx/auth
    chmod 700 /etc/nginx/auth
    chmod 400 /etc/nginx/auth/webdav

We now have to create a /etc/nginx/sites-available/example file that will contain our actual webdav configuration. This example makes a data folder stored in /var/www/ accessible.

```nginx
server {
  listen 80;
  listen [::]:80;
  server_name webdav.example.com;
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name webdav.example.com;

  root /var/www;

  ssl_certificate /etc/letsencrypt/live/www.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/www.example.com/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/www.example.com/fullchain.pem;

  ssl_dhparam /etc/nginx/ssl/dhparam.pem;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
  ssl_prefer_server_ciphers on;

  add_header Strict-Transport-Security max-age=15768000;
  add_header X-Frame-Options DENY;
  add_header X-Content-Type-Options nosniff;

  location /data {
    autoindex on;
    autoindex_exact_size off;
    client_body_temp_path /tmp;
    dav_methods PUT DELETE MKCOL COPY MOVE;
    #dav_ext_methods PROPFIND OPTIONS;
    create_full_put_path on;
    dav_access user:r group:r;

    auth_basic "Restricted access";
    auth_basic_user_file /etc/nginx/auth/webdav;
  }

}
```

The last thing we have to do is to create a symlink so that nginx will load our configuration :

    ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/example

Like before, let’s make sure our configuration is correct and then reload the daemon :

    nginx -t
    nginx -s reload

That’s it for the WebDAV configuration server-side !
