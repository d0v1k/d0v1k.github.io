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

### Let's Encrypt

Let's create a configuration file for letsencrypt:

    mkdir /etc/letsencrypt

```
echo 'rsa-key-size = 3072
renew-by-default
text = True
agree-tos = True
renew-by-default = True
authenticator = webroot
email = admin@example.com
webroot-path = /var/www/letsencrypt/' > /etc/letsencrypt/cli.ini
```

We also need to create the directory structure where letsencrypt ACME challenge temporary files will be stored :

    mkdir -p /var/www/letsencrypt/.well-known

### Nginx configuration

We now need to configure nginx by adding the following in the /etc/nginx/sites-available/default file, anywhere in the server{} block that is configured to listen on port 80.

```
location /.well-known/acme-challenge {
  root /var/www/letsencrypt;
}
```

Let's make sure that we haven't done anything wrong :

    nginx -t

The command should give you the following output :

    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful

If that's the case, you can safely reload the nginx daemon :

    nginx -s reload 

### Certificate request

Now that letsencrypt and nginx are properly configured, we can request our certificate from letsencrypt :

    letsencrypt --config /etc/letsencrypt/cli.ini certonly -w /var/www/letsencrypt -d www.example.com

Please do modify www.example.com by your server's FQDN, and please note that the letsencrypt servers need to be able to resolve that name to your server's IP.

If everything goes well, your certificates will be generated and stored in the /etc/letsencrypt folder.

### WebDAV configuration

Now that we've obtained our certificate from letsencrypt, we can begin configuring nginx.

First, we need to comment two SSL directives from the default nginx configuration :

    sed -i '/ssl_/ s/^/#/' /etc/nginx/nginx.conf

The the following content to your server block:

```
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;

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

ssl_stapling on;
ssl_stapling_verify on;
resolver 127.0.0.1 valid=300s;
resolver_timeout 5s;
```

We now need to generate a dhparam.pem file :

    mkdir /etc/nginx/ssl && chmod 700 /etc/nginx/ssl
    openssl dhparam -out /etc/nginx/ssl/dhparam.pem 3072
    chmod 600 /etc/nginx/ssl/dhparam.pem

Let's now generate a HTTP basic authentication file. This example creates a user named example :

    mkdir /etc/nginx/auth

    htpasswd -c /etc/nginx/auth/webdav example
    New password: 
    Re-type new password: 
    Adding password for user user

This file has to be readable by the user running your webserver. For security reasons, we'll make it readable only by him :

    chown -R www-data:nogroup /etc/nginx/auth
    chmod 700 /etc/nginx/auth
    chmod 400 /etc/nginx/auth/webdav

We now have to create a /etc/nginx/sites-available/example file that will contain our actual webdav configuration. This example makes a data folder stored in /var/www/ accessible.

```
server {
  listen 80;
  listen [::]:80;
  server_name www.example.com;
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name www.example.com;

  root /var/www;

  location / {
    index index.html;
  }

  location /.well-known/acme-challenge {
    root /var/www/letsencrypt;
  }

  location /data {
    client_body_temp_path /tmp;
    dav_methods PUT DELETE MKCOL COPY MOVE;
    dav_ext_methods PROPFIND OPTIONS;
    create_full_put_path on;
    dav_access user:r group:r;

    auth_basic "Restricted access";
    auth_basic_user_file auth/webdav;

    limit_except GET {
      allow <YOUR IP HERE>;
      deny all;
    }
  }

}
```

The last thing we have to do is to create a symlink so that nginx will load our configuration :

    ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/example

Like before, let's make sure our configuration is correct and then reload the daemon :

    nginx -t
    nginx -s reload

That's it for the WebDAV configuration server-side !
