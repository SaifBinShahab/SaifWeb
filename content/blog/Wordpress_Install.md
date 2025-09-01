+++
title = "Setting up Wordpress"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "DevOps", "Homelab", "Server"]
+++
## Priliminary Setup

### Set hostname
```sh
sudo vim /etc/hostname
```

For example the put the following line as hostname.
```sh
myserver
```
&ensp;

### Setup /etc/hosts
```sh
sudo vim /etc/hosts
```

For example the put the following line as hosts.
```sh
127.0.0.1 localhost
:: localhost
127.0.1.1 wordpress mywebsite.com
```
&ensp;

### Create New User
```sh
sudo useradd -m -g users -G sys,adm -s /bin/bash dev

sudo passwd dev
```
&ensp;

### Add the user to `sudo`
```sh
sudo usermod -aG sudo dev
```
&ensp;

## Wordpress Setup

## Install packages
```sh
sudo apt install -y mariadb-server apache2 apache2-utils unzip imagemagick libmagickcore-6.q16-6-extra libapache2-mod-php php-imagick php-curl php-gd php-intl php-mbstring php-mysql php-soap php-xml php-zip
```

- You can also install `php-xmlrpc` if you need. But it is best to not install this one.

- Check `apache2` status.
```sh
sudo systemctl status apache2
```
&ensp;

## Setup MariaDB
- Check `mariadb` status.
```sh
sudo systemctl status mariadb # It should be active and running.
```

- Securing mariadb. 
```sh
sudo mysql_secure_installation
```

For the `unix_socket authentication` we can answer `no`. That will enable standard authentication and will work just fine.

- Login as database root user.
```sh
sudo mariadb -u root -p
```

- Create new user for `wordpress`.
```sql
CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'password';
```

- Create a database named `db`.
```sql
CREATE DATABASE IF NOT EXISTS db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

- Check the created database.
```sql
SHOW DATABASES;
```

- Grant priviliges to the new user.
```sql
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress'@'localhost';
```

- Flush and save changes.
```sql
FLUSH PRIVILEGES;
```

- Quit mariadb.
```sql
quit;
```

## Download and Extract WordPress
```sh
cd /var/www

sudo wget https://wordpress.org/latest.zip

sudo unzip latest.zip

sudo chown -R www-data:www-data wordpress
```
&ensp;

## Add a configuration file for WordPress to Apache

- Create new `.conf` file for wordpress.
```sh
sudo vim /etc/apache2/sites-available/wordpress.conf
```

- Add the following lines in the `.conf` file
```xml
<VirtualHost *:80>
    DocumentRoot /var/www/wordpress
    <Directory /var/www/wordpress>
        Options FollowSymLinks
        AllowOverride All
        DirectoryIndex index.php
        Require all granted
    </Directory>
    <Directory /var/www/wordpress/wp-content>
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
```
&ensp;

- You can also set `AllowOverride Limit Options FileInfo`. But that will create some problems when securing the `wp-login` with apache `htpasswd`.

- Disable the default site and enable the new wordpress site.
```sh
sudo a2dissite 000-default.conf

sudo a2ensite wordpress.conf
```

- Enable some `apache` modules.
```sh
sudo a2enmod rewrite
```

- Restart `apache2`.
```sh
sudo systemctl restart apache2
```
&ensp;

After all of that, access your site and then fill out form in the web browser. Then, you’re all set!
