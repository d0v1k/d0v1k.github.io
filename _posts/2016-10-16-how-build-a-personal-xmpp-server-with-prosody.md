---
published: true
layout: post
comments: true
title:  "How to build a personal XMPP Server with Prosody"
date:   2016-09-12
categories:
  - XMPP-server
  - Prosody
---

### Install Prosody

Add Prosodys repository

    echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list
 
Adding the key file
 
    wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add -

Finally to have APT find our packages, run:

    sudo apt-get update
 
Then to install the Prosody package simply run:

    apt-get install prosody lua-dbi-mysql

### Networking
 
Edit your /etc/hosts and add the following line to it:

    10.100.10.1     servername example.com pal.example.com

Save the file. That's it.
 
Verify that Prosody is running

    systemctl status prosody

Prosody also comes with its own utility for controlling the XMPP server, called prosodyctl (similar to Apache's apachectl). You can also check the status of prosody using this tool like so:
 
    prosodyctl status

Prosody is running with PID 1820

### Install Let's Encrypt

    mkdir /opt/certbot
    cd /opt/certbot
    wget https://dl.eff.org/certbot-auto
    chmod a+x ./certbot-auto

Run cerbot-auto to generate certificate

    ./certbot-auto certonly

Declaring host

    cp -a /etc/prosody/conf.avail/example.com.cfg.lua /etc/prosody/conf.avail/example.cfg.lua

Change the settings for VirtualHost and enabled so you have:
```
[..]
 VirtualHost "example.com"
         enabled = true -- Remove this line to enable this host
[..]
```

SSL certificate:

Make sure you configure Prosody to use the 'fullchain.pem' file as certificate.

For example:

```
 ssl = {
   certificate = "/etc/letsencrypt/live/example.com/fullchain.pem";
   key = "/etc/letsencrypt/live/example.com/privkey.pem"
 }
```

Note for letsencrypt users: You'll need to give permissions to prosody user to the certificates.

    chown -R root:ssl-cert /etc/letsencrypt
    chmod g+r -R /etc/letsencrypt
    chmod g+x /etc/letsencrypt/{archive,live}

Now create the symbolic link in« /etc/prosody/conf.d/ » with:

    ln -sf /etc/prosody/conf.avail/example.com.cfg.lua /etc/prosody/conf.d/example.cfg.lua

### Create users

Creating user accounts is done with the command « prosodyctl »

    prosodyctl adduser user1@example.com

### Install Prosody modules

Clone Repository

    cd /opt/
    hg clone https://hg.prosody.im/prosody-modules/ prosody-modules

Install Modules

Useful Modules (Mobile support)

```
cp -v /opt/prosody-modules/mod_carbons/mod_carbons.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_smacks/mod_smacks.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_smacks_offline/mod_smacks_offline.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_csi/mod_csi.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_filter_chatstates/mod_filter_chatstates.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_throttle_presence/mod_throttle_presence.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_http_upload/mod_http_upload.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_blocking/mod_blocking.lua /usr/lib/prosody/modules/
cp -v /opt/prosody-modules/mod_roster_allinall/mod_roster_allinall.lua /usr/lib/prosody/modules/
```

Add to /etc/prosody/prosody.cfg.lua

```
                "groups"; -- Enable Shared roster support 
                "carbons";
                "smacks";
                "smacks_offline";
                "csi";
                "filter_chatstates";
                "throttle_presence";
                "http_upload";
                "blocking";
                "roster_allinall";
```

Restart Prosody

    prosodyctl restart
