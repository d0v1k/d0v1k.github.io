---
published: true
layout: post
comments: true
title:  "How To Install Odoo 9 on Ubuntu-16.04"
date:   2016-09-12
categories:
  - Howtos
  - blog
---
Update Ubuntu Repository

`sudo apt-get update`
`sudo apt-get upgrade -y`
 
Install PostgreSQL Server

	sudo apt-get install postgresql -y
 
Creating the ODOO PostgreSQL User

	sudo su - postgres -c "createuser -s odoo"
	sudo systemctl status postgresql
 
Install Dependencies

Install tool packages

	sudo apt-get install wget subversion git bzr bzrtools python-pip gdebi-core -y
 
Install python packages

	sudo apt-get install python-dateutil python-feedparser python-ldap python-libxslt1 python-lxml python-mako python-openid python-psycopg2 python-pybabel python-pychart python-pydot python-pyparsing python-reportlab python-simplejson python-tz python-vatnumber python-vobject python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-docutils python-psutil python-mock python-unittest2 python-jinja2 python-pypdf python-decorator python-requests python-passlib python-pil -y
 
Install python dev/dependencies packages
 
	sudo apt-get install libpq-dev libxml2-dev libxslt1-dev python-dev python-dev libldap2-dev libsasl2-dev libssl-dev -y
	sudo -H pip install --upgrade pip

Install python libraries

	sudo -H pip install gdata psycogreen ofxparse
	sudo -H pip install suds

Install other required packages
 
	sudo apt-get install node-clean-css -y
	sudo apt-get install node-less -y
	sudo apt-get install python-gevent -y

Install wkhtml and place shortcuts on correct place for ODOO 9

	wget http://download.gna.org/wkhtmltopdf/0.12/0.12.1/wkhtmltox-0.12.1_linux-trusty-amd64.deb
	sudo gdebi --n wkhtmltox-0.12.1_linux-trusty-amd64.deb
	sudo ln -s /usr/local/bin/wkhtmltopdf /usr/bin
	sudo ln -s /usr/local/bin/wkhtmltoimage /usr/bin

Install ODOO

	sudo adduser --system --shell=/bin/bash --home=/opt/odoo --gecos 'ODOO' --group odoo
	sudo passwd odoo

The user should also be added to the sudo'ers group.

	sudo adduser odoo sudo 
 
Create Log directory
 
	sudo mkdir /var/log/odoo
	sudo chown odoo:odoo /var/log/odoo

Installing ODOO Server
 
	sudo git clone --depth 1 --branch 9.0 https://www.github.com/odoo/odoo /opt/odoo

Installing libraries
 
	sudo apt-get install nodejs npm
	sudo npm install -g less
	sudo npm install -g less-plugin-clean-css

Create symlink for node

	sudo ln -s /usr/bin/nodejs /usr/bin/node
 
Create custom module directory

	sudo su odoo -c "mkdir /opt/odoo/custom"
	sudo su odoo -c "mkdir /opt/odoo/custom/addons"

Setting permissions on home folder

	sudo chown -R odoo:odoo /opt/odoo/* 
 
Create server config file

	sudo mkdir /etc/odoo
	sudo chown odoo:odoo /etc/odoo
	sudo cp /opt/odoo/debian/openerp-server.conf /etc/odoo/odoo-server.conf
	sudo chown odoo:odoo /etc/odoo/odoo-server.conf
	sudo chmod 640 /etc/odoo/odoo-server.conf
 
Change server config file

	sudo sed -i s/"db_user = .*"/"db_user = odoo"/g /etc/odoo/odoo-server.conf
	sudo sed -i s/"; admin_passwd.*"/"admin_passwd = MY768WFUNtkT@"/g /etc/odoo/odoo-server.conf
	sudo sed -i '/addons_path/d' /etc/odoo/odoo-server.conf
	sudo su root -c "echo 'logfile = /var/log/odoo/odoo-server.log' >> /etc/odoo/odoo-server.conf"
	sudo su root -c "echo 'data_dir = /opt/odoo/data' >> /etc/odoo/odoo-server.conf"
	sudo su root -c "echo 'addons_path=/opt/odoo/addons,/opt/odoo/custom/addons' >> /etc/odoo/odoo-server.conf"
 
Create startup file

Add to /opt/odoo/start.sh

```
#!/bin/sh
/opt/odoo/openerp-server --config=/etc/odoo/odoo-server.conf
 
sudo chmod 755 /opt/odoo/start.sh
```

Adding ODOO as a deamon

{% highlight bash %}
cat <<EOF > ~/odoo-server
#!/bin/sh
#Short-Description:EnterpriseBusinessApplications
#Description:ODOOCommunityApplications
###ENDINITINFO
PATH=/bin:/sbin:/usr/bin
DAEMON=/opt/odoo/openerp-server
NAME=odoo-server
DESC=odoo-server

#Specifytheusername(Default:odoo).
USER=odoo

#Specifyanalternateconfigfile(Default:/etc/openerp-server.conf).
CONFIGFILE="/etc/odoo/odoo-server.conf"

#pidfile
PIDFILE=/var/run/\${NAME}.pid

#AdditionaloptionsthatarepassedtotheDaemon.
DAEMON_OPTS="-c\$CONFIGFILE"
[-x\$DAEMON]||exit0
[-f\$CONFIGFILE]||exit0
checkpid(){
[-f\$PIDFILE]||return1
pid=\`cat\$PIDFILE\`
[-d/proc/\$pid]&&return0
return1
}

case"\${1}"in
start)
echo-n"Starting\${DESC}:"
start-stop-daemon--start--quiet--pidfile\$PIDFILE\
--chuid\$USER--background--make-pidfile\
--exec\$DAEMON--\$DAEMON_OPTS
echo"\${NAME}."
;;
stop)
echo-n"Stopping\${DESC}:"
start-stop-daemon--stop--quiet--pidfile\$PIDFILE\
--oknodo
echo"\${NAME}."
;;

restart|force-reload)
echo-n"Restarting\${DESC}:"
start-stop-daemon--stop--quiet--pidfile\$PIDFILE\
--oknodo
sleep1
start-stop-daemon--start--quiet--pidfile\$PIDFILE\
--chuid\$USER--background--make-pidfile\
--exec\$DAEMON--\$DAEMON_OPTS
echo"\${NAME}."
;;
*)
N=/etc/init.d/\$NAME
echo"Usage:\$NAME{start|stop|restart|force-reload}">&2
exit1
;;

esac
exit0
EOF
{% endhighlight %}

Security Init File

	sudo mv ~/odoo-server /etc/init.d/odoo-server
	sudo chmod 755 /etc/init.d/odoo-server
	sudo chown root: /etc/init.d/odoo-server

Start ODOO on Startup

	sudo update-rc.d odoo-server defaults

Starting Odoo Service"

	sudo su root -c "/etc/init.d/odoo-server start"
