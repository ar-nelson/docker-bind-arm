docker-bind
===========

ISC Bind server for general purpose internal name server with webmin for easy management.

### TL;DR ###

Make the persistent directories
    mkdir -p /srv/bind/named
    mkdir /srv/bind/zones
    mkdir /srv/bind/webmin

Here is a sample command using all the options.
    docker run -d \
    -p 53:53 -p 53:53/udp \
    -p 10000:10000 \
    -v /srv/bind/named:/etc/named \
    -v /srv/bind/zones:/var/lib/bind \
    -v /srv/bind/webmin:/etc/webmin \
    -e PASS=newpass \
    -e NET=172.17.0.0\;192.168.0.0\;10.1.2.0 \
    --name bind --hostname bind \
    cosmicq/docker-bind

Log into webmin and manage your server
    http://hostname.or.ip:10000
    (root:newpass)

### Options ###

##### Ports #####

**53** - TCP functions for named.  You might not need this if you are not going to transfer
 zone files or anything.

**53/udp** - This is the bulk of the DNS lookups.

**10000** - This is for the webmin access so you can use a web interface to modify DNS.

##### Volumes #####

If you want any kind of persistence for DNS, that is if you want your information to survive
 reboots or anything, you might like to store it outside the container.  This also allows for
 easy backups and importing zone files.

I like to make all my external volumes on /srv/containername/volume so that is what is in the
 example.  You are free to change that to whatever makes you happy.

**/srv/bind/named:/etc/named** - Location of named configuration files.

**/srv/bind/named:/var/lib/bind** - Location of named zone files.

**/srv/bind/webmin:/etc/webmin** - Location of webmin configuration files and plugins.

##### Environment Variables #####

**PASS** - This is used to set the root password which is primarily used for access webmin

**NET** - By default, webmin allows all IP addresses to access it.  By adding IP addresses
 you are restricting access to webmin.  You can add multiple IP addresses or ranges. 
 just separate them with a backslash semicolon. 

### Using Webmin ###

You should probably follow the guide at webmin.com
    http://doxfer.webmin.com/Webmin/BINDDNSServer

For a quick, just get me started guide:
    Click on Servers -> BIND DNS Server
    Under "Existing DNS Zones", click on "Create master zone"
    Enter "Domain name / Network" (example: test.lab)
    Enter "Email address" (admin@test.lab)

That should be enough to create your first zone.

### Editing the config files by hand ###

If you add or edit the config files in /srv/bind/named by hand, you need to restart the named
process for that change to take effect.  This uses phusion/baseimage which runs "runit" to
start services.  If the service dies, runit will start it again.  All we need to do to restart
a process is to kill it and it will start right back up again.
    Click on System -> Running processes
    Click on the process ID for /usr/sbin/named
    Click on the Kill button and the process will simply restart
