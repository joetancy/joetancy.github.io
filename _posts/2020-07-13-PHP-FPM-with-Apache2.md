---
title: "PHP-FPM with Apache2"
date: 2020-07-13T17:23:30-04:00
categories:
  - blog
---

Processing PHP is slow and clunky on Apache Modules, therefore people move to NGINX for higher HTTP response concurrency.

Apache has something similar called mpm_events, to use this we need to disable the Apache PHP module.

Below are the steps for a complete change from Apache PHP module to PHP-FPM for processing, and optimising Apache to use mpm_events instead of mpm_prefork.

First, we need to install PHP-FPM, configure Apache to route .php processing to PHP-FPM, then optimise the number of PHP-FPM threads. After which we will change Apache from mpm_prefork to mpm_events for higher concurrency processing.

This guide assumes you are operating on Ubuntu 18.04 LTS

### Installing PHP-FPM

```bash
sudo apt install python-software-properties 
sudo add-apt-repository ppa:ondrej/php
apt update
sudo apt install php7.3 php7.3-fpm php7.3-curl php7.3-mysql php7.3-xmlrpc php7.3-bcmath php7.3-mbstring php7.3-xml
```

Now we have to check if PHP-FPM is running as a daemon.

```bash
sudo systemctl status php7.3-fpm
```

You should be able to see something like this.

```bash
php7.3-fpm.service - The PHP 7.3 FastCGI Process Manager Loaded: loaded (/lib/systemd/system/php7.3-fpm.service;
enabled; vendor preset: enabled)
Active: active (running) since Tue 2019-07-23 03:28:26 UTC; 17min ago
Docs: man:php-fpm7.3(8)
Main PID: 30011 (php-fpm7.3)
Status: "Processes active: 38, idle: 3, Requests: 20367, slow:
0, Traffic: 17.2req/sec"

Tasks: 102 (limit: 4915)
CGroup: /system.slice/php7.3-fpm.service
        ├─10217 php-fpm: pool www 
        ├─10277 php-fpm: pool www 
        ├─10278 php-fpm: pool www 
        └─30011 php-fpm: master process
(/etc/php/7.3/fpm/php-fpm.conf)

Jul 23 03:28:26 systemd[1]: Starting The PHP 7.3 FastCGI Process Manager...
Jul 23 03:28:26 systemd[1]: Started The PHP 7.3 FastCGI Process Manager.
```

### Configuring Apache2

Enable these modules so that Apache can route PHP processing to PHP-FPM.

```bash
a2enmod actions alias proxy_fcgi setenvif
```

Edit the Apache configuration file to route the processing. (Also edit for 000-default-ssl.conf if needed)

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Add these lines into this file

```bash
<FilesMatch \.php$> 
    SetHandler
"proxy:unix:/run/php/php7.3-fpm.sock|fcgi://localhost/" 
</FilesMatch>
```
Make sure your .sock is in the correct path

Restart Apache to test if your website still works.

```bash
sudo systemctl restart apache2
```

You may have to run these commands to enable PHP-FPM with Apache

```bash
a2enmod proxy_fcgi setenvif 
a2enconf php7.3-fpm
```

### Changing mpm module

Now we change the mpm module from prefork to events.

```bash
a2dismod php7.3 mpm_prefork 
a2enmod mpm_event
sudo systemctl restart apache2
```

Check if Server MPM is changed to events

```bash
sudo apachectl -V | grep MPM
```

You should see

```bash
Server MPM: event
```

### Optimising Apache2 mpm_event

```bash
cd /etc/apache2/mods-enabled/
```

```bash
nano mpm_event.conf
```

```bash
<IfModule mpm_event_module> 
    KeepAlive On
    KeepAliveTimeout 5 
    MaxKeepAliveRequests 128

    ServerLimit 10
    StartServers 4
    ThreadLimit 128 
    ThreadsPerChild 128 
    MinSpareThreads 256 
    MaxSpareThreads 512 
    MaxRequestWorkers 1280 
    MaxConnectionsPerChild 2048
</IfModule>
```

### Optimising PHP-FPM

Now that we have increased the concurrency of Apache2, we now have to increase the number of PHP processing servers.

```bash
cd /etc/php/7.3/fpm/pool.d/ 
nano ​www.conf
```

Make sure the following is entered

```bash
pm = dynamic
pm.max_children = 100 
pm.start_servers = 20 
pm.min_spare_servers = 10 
pm.max_spare_servers = 30 
pm.process_idle_timeout = 10s;
```

### Restart both process

```bash
sudo systemctl restart php7.3-fpm 
sudo systemctl restart apache2
```

And you're done! Congratulations! PHP processing is now handled by PHP-FPM!