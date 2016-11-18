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

### Setting up DKIM

We are going to use Postfix to send mail.

Remove Sendmail and install Postfix

    sudo apt purge sendmail && sudo apt install postfix

Install OpenDKIM

    sudo apt install opendkim opendkim-tools
 
Then add postfix user to opendkim group

    sudo gpasswd -a postfix opendkim

Edit OpenDKIM main configuration file.

    sudo vi /etc/opendkim.conf

Add the following lines below.

Uncomment the following lines. Replace simple with <b>relaxed/simple</b>.

    Canonicalization   simple
    Mode               sv
    SubDomains         no
  
Then add the following lines below below <b>SubDomains  no</b>.

    AutoRestart         yes
    AutoRestartRate     10/1M
    Background          yes
    DNSTimeout          5
    SignatureAlgorithm  rsa-sha256
 
Add the following lines at the end of this the file.
    
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

    # This is a basic configuration that can easily be adapted to suit a standard
    # installation. For more advanced options, see opendkim.conf(5) and/or
    # /usr/share/doc/opendkim/examples/opendkim.conf.sample.
    
    # Log to syslog
    Syslog                  yes
    # Required to use local socket with MTAs that access the socket as a non-
    # privileged user (e.g. Postfix)
    UMask                   002
    
    # Sign for example.com with key in /etc/dkimkeys/dkim.key using
    # selector '2007' (e.g. 2007._domainkey.example.com)
    #Domain                 example.com
    #KeyFile                /etc/dkimkeys/dkim.key
    #Selector               2007
    
    # Commonly-used options; the commented-out versions show the defaults.
    Canonicalization        relaxed/simple
    Mode                    sv
    SubDomains              no
    AutoRestart         	yes
    AutoRestartRate     	10/1M
    Background          	yes
    DNSTimeout          	5
    SignatureAlgorithm  	rsa-sha256
    
    # Always oversign From (sign using actual From and a null From to prevent
    # malicious signatures header fields (From and/or others) between the signer
    # and the verifier.  From is oversigned by default in the Debian pacakge
    # because it is often the identity key used by reputation systems and thus
    # somewhat security sensitive.
    OversignHeaders         From
    
    ##  ResolverConfiguration filename
    ##      default (none)
    ##
    ##  Specifies a configuration file to be passed to the Unbound library that
    ##  performs DNS queries applying the DNSSEC protocol.  See the Unbound
    ##  documentation at http://unbound.net for the expected content of this file.
    ##  The results of using this and the TrustAnchorFile setting at the same
    ##  time are undefined.
    ##  In Debian, /etc/unbound/unbound.conf is shipped as part of the Suggested
    ##  unbound package
    
    # ResolverConfiguration     /etc/unbound/unbound.conf
    
    ##  TrustAnchorFile filename
    ##      default (none)
    ##
    ## Specifies a file from which trust anchor data should be read when doing
    ## DNS queries and applying the DNSSEC protocol.  See the Unbound documentation
    ## at http://unbound.net for the expected format of this file.
    
    TrustAnchorFile       /usr/share/dns/root.key
    
    #OpenDKIM user
    # Remember to add user postfix to group opendkim
    UserID             opendkim
    
    # Map domains in From addresses to keys used to sign messages
    KeyTable           /etc/opendkim/key.table
    SigningTable       refile:/etc/opendkim/signing.table
    
    # Hosts to ignore when verifying signatures
    ExternalIgnoreList  /etc/opendkim/trusted.hosts
    InternalHosts       /etc/opendkim/trusted.hosts

Save and close the file.

### Create Signing table, key table and trusted hosts file

Create a directory structure for OpenDKIM

    sudo mkdir /etc/opendkim
    
    sudo mkdir /etc/opendkim/keys

Change owner from root to opendkim and make sure only opendkim user can read and write to the keys directory.

    sudo chown -R opendkim:opendkim /etc/opendkim
    
    sudo chmod go-rw /etc/opendkim/keys
    
Create the signing table.

    sudo nano /etc/opendkim/signing.table

Add this line to the file. Replace your-domain.com with your real domain.

    *@your-domain.com default._domainkey.your-domain
    
Then create the key table.

    sudo vi /etc/opendkim/key.table

Add the following line. Replace your-domain with your actual domain.

    default._domainkey.your-domain    your-domain.com:default:/etc/opendkim/keys/your-domain.com/default.private

Save and close the file.

Configure Trusted Hosts

Create the file.

    sudo vi /etc/opendkim/trusted.hosts

Add the following lines to the newly created file.

    127.0.0.1
    localhost
    
    *.your-domain.com
    
The above means that messages coming from the above IP addresses and domains will be trusted and signed.

### Generate Private/Public Keypair

Since DKIM is used to sign outgoing messages and verify incoming messages, we need to generate a private key for signing and a public key for remote verifier. Public key will be published in DNS.

Create a separate folder for the domain.

    sudo mkdir /etc/opendkim/keys/your-domain.com

Generate keys using opendkim-genkey tool.

    sudo opendkim-genkey -b 2048 -d your-domain.com -D /etc/opendkim/keys/your-domain.com -s default -v
