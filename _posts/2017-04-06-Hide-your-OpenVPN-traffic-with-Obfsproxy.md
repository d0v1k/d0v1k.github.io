---
published: true
layout: post
comments: true
title:  "How to install OpenVPN & Obfsproxy on Ubuntu 16.04 LTS"
date:   2017-04-06
categories:
  - Howtos
  - OpenVPN
---

###  Configure ufw

Set UFW to allow SSH:

   sudo ufw allow ssh

We use OpenVPN over TCP, so UFW must also allow TCP traffic over port 443.

   sudo ufw allow 443/tcp

Allow traffic over port 21194 for obfsproxy 

   sudo ufw allow 21194/tcp

The UFW forwarding policy needs to be set as well. We'll do this in the primary configuration file.

   sudo vi /etc/default/ufw

Make the following change:

   DEFAULT_FORWARD_POLICY="ACCEPT"

Save and exit.

Next we will add additional UFW rules for network address translation and IP masquerading of connected clients.

   sudo  vi /etc/ufw/before.rules

 Next, add the area in Bold for OPENVPN RULES:
<pre> 
 #
 # rules.before
 #
 # Rules that should be run before the ufw command line added rules. Custom
 # rules should be added to one of these chains:
 #   ufw-before-input
 #   ufw-before-output
 #   ufw-before-forward
 #
 
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES
</pre> 
Save and exit.

With the changes made to UFW, we can now enable it. Enter into the command prompt:

   ufw enable

To check UFW's primary firewall rules:

   ufw status
