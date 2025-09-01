+++
title = "Securing Apache Webserver"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
##  Disable directory listing.
We can do this in 2 different ways one is to configure `/etc/apache2/apache2.conf`. Or we can also put the config in `.htaccess` file and put the `.htaccess` file in the root of the directory we're protecting (i.e. the wordpress directory).

**First lets do this with the `.htaccess` way.**

```sh
sudo vim /var/www/wordpress/.htaccess

# Append the config below

Options -Indexes

# Make the .htaccess readonly.
sudo chmod 0444 /var/www/wordpress/.htaccess

sudo systemctl restart apache2
```
&nbsp;

**Then we'll see how to do it with apache2.conf.**
```sh
sudo vim /etc/apache2/apache2.conf

# In "<Directory /var/www/wordpress>" Append "Options -Indexes" in the config below like this.
<Directory /var/www/wordpress>
        Options -Indexes
        Options FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>

# We can also put "Options -Indexes" inside "<Directory /var/www>" to protect that directory.
```
&nbsp;

## Disable Server Signature to prevent server information leak.

To do this we can like the previous config do it in two different ways.

**First we'll see how to do it with apache2.conf.**
```sh
sudo vim /etc/apache2/apache2.conf

# In "<Directory /var/www/wordpress>" Append "Options -Indexes" in the config below like this.
<Directory /var/www/wordpress>
        Options -Indexes
        Options FollowSymLinks
        AllowOverride None
        Require all granted
        ServerSignature off
</Directory>

# We can also put "Options -Indexes" inside "<Directory /var/www>" to protect that directory.
```
&nbsp;

**Then lets do this with the `.htaccess` way.**
```sh
sudo vim /var/www/wordpress/.htaccess

# Append the config below

ServerSignature off

# Make the .htaccess readonly.
sudo chmod 0444 /var/www/wordpress/.htaccess

sudo systemctl restart apache2
```
&nbsp;

### Note: According to my experience when we're mitigating directory file listing, the `.htaccess` method works better and when we're mitigating Server Signature, then configuring `/etc/apache2/apache2.conf` works better. So, keep this in mind when securing Apache. And try first what is supposed to actually work better, then if it doesn't work as intended try the other way. InShaa Allah it will work.
&nbsp;

## Password Protect a page using apache.
For this the "apache2-utils" package is required which we should've installed during the wordpress setup. In this example we're going to protect the `wp-login.php` page which is under `wp-admin` directory.

```sh
sudo htpasswd -c /etc/apache2/.htpasswd username

sudo vim /var/www/wordpress/wp-admin/.htaccess

# Append the configs

AuthName "Admin Login"
AuthUserFile /etc/apache2/.htpasswd
AuthType Basic
Require valid-user

# Make the .htaccess readonly.
sudo chmod 0444 /var/www/wordpress/wp-admin/.htaccess

sudo systemctl restart apache2
```
&nbsp;

Done!
