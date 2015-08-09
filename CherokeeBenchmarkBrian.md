# Cherokee + Apache + Lighttpd Benchmark #

## Software ##
  * cherokee 0.6.0 beta2
  * apache 2.0.59
  * lighttpd 1.4.16

## Hardware ##
  * 733 MHz PIII
  * 256 MB RAM
  * 80GB 7200RPM IDE HD
  * Debian GNU/Linux 4.0

## Background ##

I installed a fresh installation of Debian on the server hardware. Right after you login you will need to get sudo to perform root commands from your account:

```
su
apt-get install sudo
```

Then add yourself to the `/etc/sudoers` file by running `visudo` and adding yourself in the user section. I just followed the root entry as this does not need to be a very secure server since it will not be running publicly. Now make sure you get back to your account and do:

```
sudo apt-get install gcc make automake autoconf libtool
mkdir src ; cd src
sudo mkdir /usr/local/cherokee
sudo mkdir /usr/local/lighttpd
```

The installed version of gcc is 4.1.2

## Cherokee Setup Details ##

The following is what I executed to build Cherokee:

```
wget http://www.cherokee-project.com/download/0.6/0.6.0/cherokee-0.6.0b863.tar.gz
tar zxvf cherokee-0.6.0b863.tar.gz
cd cherokee-0.6.0b863
./configure --prefix=/usr/local/cherokee/0.6.0b863
make
sudo make install
```

Here is the configuration for cherokee:

```
server!port = 80
server!timeout = 60
server!keepalive = 1
server!keepalive_max_requests = 500
server!pid_file = /var/run/cherokee.pid
server!server_tokens = full
server!encoder!gzip!allow = html,html,txt
server!panic_action = /usr/local/cherokee/0.6.0b863/bin/cherokee-panic
server!mime_files = /usr/local/cherokee/0.6.0b863/etc/cherokee/mime.types

vserver!default!document_root = /usr/local/cherokee/0.6.0b863/var/www
vserver!default!directory_index = index.html

vserver!default!directory!/!handler = common
vserver!default!directory!/!handler!iocache = 1
vserver!default!directory!/!priority = 1
```

To run the web server I used:

```
cd /usr/local/cherokee/0.6.0b863
sudo sbin/cherokee -C etc/cherokee/cherokee.conf
```

## Apache Setup Details ##

The following is what I executed to build Apache:

```
wget http://apache.oregonstate.edu/httpd/httpd-2.0.59.tar.gz
tar zxvf httpd-2.0.59.tar.gz
cd httpd-2.0.59
./configure --prefix=/usr/local/apache/2.0.59
make
sudo make install
```

I used the supplied `highperformance.conf` configuration file. I started the server with:

```
cd /usr/local/apache/2.0.59
sudo bin/httpd -k start -f conf/highperformance.conf
```

The server ran using prefork.

## Lighttpd Setup Details ##

The following is what I executed to build lighttpd:

```
wget http://www.lighttpd.net/download/lighttpd-1.4.16.tar.gz
tar zxvf lighttpd-1.4.16.tar.gz
cd lighttpd-1.4.16
./configure --prefix=/usr/local/lighttpd/1.4.16
make
sudo make install
```

The configuration I used looked like this:

```
server.modules = (
    "mod_access",
    "mod_accesslog"
)

server.document-root = "/var/www"

mimetype.assign = (
    ".html" => "text/html",
    ".txt" => "text/plain"
)
```

I started the server with:

```
cd /usr/local/lighttpd/1.4.16
sudo sbin/lighttpd -f sbin/lighttpd.conf
```

## Benchmark ##

I will perform several different benchmarks on each webserver. This is to help gauge what type of performance each server can handle in the different conditions. Each test will have SSL turned on and turned off.

### small static file test ###
  * filesize: 99 bytes
  * command: `ab -c 2 -t 2 -k http://localhost/index0.html`

### large static file test ###
  * filesize: 1.5MB
  * command: `ab -c 2 -t 2 -k http://localhost/static.txt`

## Results ##

I have included cherokee with both iocaching on and off. The out of the box setting is that iocache is turned on.

### small static file test w/ keepalive ###
  * cherokee 0.6.0b863 w/ iocache - 7816 reqs./sec.
  * cherokee 0.6.0b863 w/o iocache - 5761 reqs./sec.
  * lighttpd 1.4.16 - 4884 reqs./sec.
  * apache 2.0.59 - 2924 reqs./sec.

![http://www.cherokee-project.com/images/benchmark1.png](http://www.cherokee-project.com/images/benchmark1.png)

### small static file test w/o keepalive ###
  * cherokee 0.6.0b863 w/ iocache - 2182 reqs./sec.
  * cherokee 0.6.0b863 w/o iocache - 1874 reqs./sec.
  * lighttpd 1.4.16 - 2255 reqs./sec.
  * apache 2.0.59 - 1250 reqs./sec.

### large static file test w/ keepalive ###
  * cherokee 0.6.0b863 w/ iocache - 108 reqs./sec.
  * cherokee 0.6.0b863 w/o iocache - 107 reqs./sec.
  * lighttpd 1.4.16 - 106 reqs./sec.
  * apache 2.0.59 - 94 reqs./sec.

![http://www.cherokee-project.com/images/benchmark2.png](http://www.cherokee-project.com/images/benchmark2.png)

### large static file test w/o keepalive ###
  * cherokee 0.6.0b863 w/ iocache - 88 reqs./sec.
  * cherokee 0.6.0b863 w/o iocache - 88 reqs./sec.
  * lighttpd 1.4.16 - 92 reqs./sec.
  * apache 2.0.59 - 118 reqs./sec.

## Conclusion ##

TODO: write something here

If you feel I need to supply any more detail about any part of this process, please let me know. I want this test to be as accurate as possible. I also would like to continue using this process to test future releases of each software and ultimately help improve them. I am a Cherokee developer, but if I notice anything that would help other webservers I am more than happy to let that community know.


