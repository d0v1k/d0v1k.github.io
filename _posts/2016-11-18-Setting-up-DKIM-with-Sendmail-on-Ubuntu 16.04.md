---
published: true
layout: post
comments: true
title:  "Setting up DKIM with Postfix on Ubuntu 16.04"
date:   2016-11-18
categories:
  - Howtos
  - Email Server
---

We are going to use Postfix to send mail.

#### Remove Sendmail and install Postfix

    sudo apt purge sendmail && sudo apt install postfix

#### Install OpenDKIM

    sudo apt install opendkim opendkim-tools
 
#### Then add postfix user to opendkim group

    sudo gpasswd -a postfix opendkim

#### Edit OpenDKIM main configuration file.

    sudo vi /etc/opendkim.conf

#### Add the following lines below.

#### Uncomment the following lines. Replace simple with <b>relaxed/simple</b>.

    Canonicalization   simple
    Mode               sv
    SubDomains         no
 
#### Add the following lines at the end of this the file.
    
    #OpenDKIM user
    # Remember to add user postfix to group opendkim
    UserID             opendkim
     
    # Map domains in From addresses to keys used to sign messages
    KeyTable           /etc/opendkim/key.table
    SigningTable       refile:/etc/opendkim/signing.table
     
    # Hosts to ignore when verifying signatures
    ExternalIgnoreList  /etc/opendkim/trusted.hosts
    InternalHosts       /etc/opendkim/trusted.hosts
    
The final configuration file is as follows:
