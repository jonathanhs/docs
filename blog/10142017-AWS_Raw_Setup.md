# AWS Raw Setup

Some of my personal AWS raw setup.

### Create new instance in AWS

1. launch instance
2. create security group
3. create key pairs and download it
4. create elastic IP and associate that IP with the instance that we created
5. update Route 53 with our own domain and elastic IP that we just created (A type *domain.com* and A type *www.domain.com*)
6. refresh DNS cache through [http://cachecheck.opendns.com/](http://cachecheck.opendns.com/)

### SSH to machine

```shell
> chmod 400 key.pem
> ssh -i key.pem ubuntu@domain.com
```

### Setup LAMP stack in the machine

```shell
> sudo apt-get install tasksel
> sudo apt-get update
> sudo tasksel install lamp-server
```

### Create new user

```shell
> sudo useradd -d /home/user -m user
> sudo passwd user
```

### Delete user

```shell
> sudo deluser user && sudo rm -r /home/user
```

### Add user to "sudo" group

```shell
> sudo usermod -a -G sudo user
```

### Set user to have bash terminal

```shell
> sudo usermod -s /bin/bash user
```

### Set password authentication to "yes"

```shell
> sudo vi /etc/ssh/sshd_config
> # change password authentication to yes
> sudo service ssh restart
```

### Create user for MySQL and grant access for user

```shell
> sudo vi /etc/mysql/my.cnf
> # set bind-address to 0.0.0.0
> sudo service mysql restart
```

Add MySQL inbound rule in AWS security group and then create user.

```mysql
# MySQL
> CREATE USER 'user'@'%' IDENTIFIED BY 'password';
> GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';
> FLUSH PRIVILEGES;
```

### Setup Git

```shell
> sudo apt-get install git
> sudo apt-get install bash-completion
```

### Setup Memcached

```shell
> sudo apt-get install memcached
> sudo apt-get install php5-memcache
> sudo apt-get install php5-memcached
> sudo apt-get install php-pear
```

### Setup Mcrypt

```shell
> sudo apt-get install php5-mcrypt
> sudo php5enmod mcrypt
> sudo service apache2 restart
```

### Setup mod_rewrite

```shell
> sudo a2enmod rewrite
> sudo service apache2 restart
```

### Create project in /var/www

```shell
> sudo adduser user www-data
> sudo chgrp www-data /var/www
> sudo chmod 775 /var/www
> sudo chown -R www-data:www-data /var/www
```
------

