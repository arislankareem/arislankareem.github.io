---
title: Status page by CachetHQ and Zabbix
layout: post
description: Documentation to Integrate the Zabbix IT service with the Cachet Status
  page
img: cachet.png
---

# Status Page using CachetHQ

### Requirements
1. CachetHQ
2. Zabbix Server
3. Python 2.7+
4. Zabbix-Cachet

### Environment
1. Remote Zabbix Server
2. Independent Cachet server, Centos 7+
3. Cachet server should support python2.7+, docker

## Information

We are achieving the status page out by feeding the Zabbix server IT service functionality to the CachetHQ. This synchronization is achieved using a python script called Zabbix-Cachet. 

There are multiple methods to run the Zabbix-Cachet, which will be mentioned on the go.

There is official Zabbix-Cachet PPA: so we can go for debian based distribution instead of centos. 

## Installation of CachetHQ
#### Pre-requisites

1. Setup a LAMP Server with latest versions.

2. Disable SELinux

3. Create a Database and Database user for Cachet

---
### Installing CachetHQ

1. Install Composer

```bash
curl -sS https://getcomposer.org/installer | php

mv composer.phar /usr/bin/composer

cd /var/www # Or wherever you chose to install web applications to

$ git clone https://github.com/cachethq/Cachet.git

$ cd Cachet

$ git tag -l
```

Always use the latest tag of Cachet. 
```bash
$ git checkout v2.3.9
```

2. Now add the database details in to the .env file

you can see a  .env.example please rename the same to .env 

add the Database details in the database sections. 

### Please note this file is used to set the mail and other Cachet functionality as per requirement. 

You will get the details how to configure mail and subscribers here:

> https://docs.cachethq.io/docs/configuring-mail

3. Execute the following commands respectively:


```bash
composer install --no-dev -o

php artisan key:generate
```

This is to generate generate an APP_KEY for encryption which you can see in .env file. 

```bash
php artisan app:install
```

4. Generate a Virtual Host, sample for Apache is mentioned below.:


```apache
<VirtualHost *:80>
    ServerName cachet.dev 
    # Or whatever you want to use
    ServerAlias cachet.dev 
    # Make this the same as ServerName
    DocumentRoot "/var/www/Cachet/public"
    <Directory "/var/www/Cachet/public">
        Require all granted 
        # Used by Apache 2.4
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```



3. Modify the Ownership of /var/www/cachet/public to the respective user. 

4. Restart the service and you can now access the Cachet using the respective URL.

5. Note down the API key from the Cachet Admin page once you have logged in. 


## Now we need to set the Zabbix IT service in the Zabbix Dashboard. 

> Configuration ->  Services

You will see root as first service and tab 

> Add child

Click the same and in this section you can name the service. 

Eg.: Shared servers as service. Please note at this time no trigger should be selected. 

Under this service Add child and here you should add the respective shared server name and corresponding triggers. 


## Installing Zabbix-Cachet

### Docker Installation

I prefer this method on latest centos as it hazel free. 

   1. Create /etc/zabbix-cachet.yml file based config-example.yml.

you can clone this repo

> https://github.com/qk4l/zabbix-cachet.git

You will get the config-example.yml

In this file only two portion needs to be touched. 

> one is Zabbix, where we have to provide the details. Please note the user should have enough privilege to access the Zabbix IT services. 

> Second is Cachet details. API token will be available from the Cachet Admin Area. 

   2. Now run docker container using the command:



   ```bash
docker run --name zabbix-cachet -v /etc/zabbix-cachet.yml:/config.yml qk4l/zabbix-cachet
   ```
	 
### Using APT

1. Add official PPA for Zabbix-Cachet:


```bash
add-apt-repository ppa:reg-tem4uk/zabbix-cachet

apt-get update
```


2. Install the package:
```bash
apt-get install zabbix-cachet
```

3. Configure the zabbix-cachet.yml
```bash
nano /etc/zabbix-cachet.yml
```

4. Restart the service:


```bash
systemctl enable zabbix-cachet

systemctl restart zabbix-cachet
```