+++
title = "Securing Wordpress"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
## 1. Setup Wordpress and continue with the following
&nbsp;

## 2. Installing SSL/TLS certificate
- A lot of the domain providers should provide an SSL certificate.
- Cloudflare privides a free SSL certificate upon setting it up as a dns. Also, when using Cloudflare make sure to use the reverse proxy service of Cloudflare and CDN and other domain and server security (like DDOS) features of Cloudflare.
- You can also install a free SSL certificate by "Let's Encrypt" which is a project of EFF. The procedure is as follows.

```sh
sudo apt install -y certbot python3-certbot-apache python3-certbot-nginx

# Install the certificate by providing the webserver technology name. We can also use "--nginx".

sudo certbot --apache

# This will detect connected domains to this server and ask to install certificates for the domains. It will allow to install certificates for multiple domains connected to the server.
```
&nbsp;

### Create a cron job to automatically renew the certificate
- The Let's Encrypt certificate stays valid for 3 months. So we need to renew it before that time period.
- We will add a cron job to renew the certificate in every 2 months.
```sh
sudo crontab -e

# Put this inside the cron file.
0 0 2 * * certbot --apache
```
&ensp;

## 3. Password Policy
Enforece a strict password policy for the users using this wordpress server if there are multiple users using the wordpress instance.

## 4. Wordpress User Management
Make sure "New User Default Role" is set to "Subscriber" or "Author". You can find it in `Settings > General > New User Default Role`.

## 4. Updates
Make sure the wordpress has the latest update.

## 5. Redundant Plugins
Delete any unnecessary plugins and pages.
&nbsp;

## 6. Setup 2FA for Wordpress.
Go to plugins and install a 2FA plugin. A good one is **"2FAS Light - Google Authenticator"**.
&nbsp;

## 7. Bruteforce Protection Plugin
Install a bruteforce protection plugin. This will protect the wordpress login page from bruteforce attacks.
A good plugin for this is **"Loginizer"**. It also provides various security recommendation.

### Using Loginizer
- Follow the recommendations provided by Loginizer.
- Loginizer also provides 2FA support.
&nbsp;

## 8. Firewall and Malware Scanner Plugin
Install "Wordfence Security - Firewall & Malware Scan".
This will allow us to setup a web application firewall which uses the OWASP web application vulnerability list and also provide mlware scanning.

### Setting up Wordfence
- In the Wordfence dashboard it will show the current security status of the wordpress site. It will be in "Learning Mode" after installation and it will try and learn from various traffic coming to the site.
- In Firewall tab we can set various settings. Under "Brute Force Protection" set the values appropriately. Make sure to set "Count failures over what time period" is set to a lower value like 30 minutes or maybe even less. Because brute force attacks are typically performed in a very short burst.
- We can also run the malware scanner maybe envery month if the site is very active.

&nbsp;

## 9. (Optional) Solving SSL/TLS issues
If we are having issues with SSL then we can install "Really Simple SSL". It will help to solve various SSL issues.
&nbsp;

## 10. Site Backup
Install a backup plugin that "clones" your entire site.
- Some good backup plugins are "Duplicator 	- Wordpress Migration Plugin". 
- Take at least weekly backups and then monthly backups. Also keep redundant backups of 1-2 months old. Or maybe even older.
- Take backups before making any changes to the website.
&nbsp;

## 11. XML-RPC Attack Prevention
Disable "XML-RPC" endpoint.
We can Install "Stop XML-RPC Attacks" plugin to completely disable XML-RPC on the site. We can also manually remove xml-rpc.

Also our server should be more secure because we didn't install or removed the "php-xmlrpc" package in the first place.
&nbsp;

## 12. Install monitoring plugin.
Install a plugin that will keep logs of the wordpress site. A good plugin is "WP Activity Log".
&nbsp;

## 13. File Permissions
Make sure the owner of the wordpress directory on the server is "www-data".

## 14. Disable File Editing

```sh
sudo vim /var/www/wordpress/wp-config.php

# Append the config below

/** DISABLE FILE EDITING */
define('DISALLOW_FILE_EDIT', true);
```
&nbsp;

## 15. Make the "wp-config.php" file readonly.
```sh
sudo chmod 0444 /var/www/wordpress/wp-config.php
```

We will still be able to edit the file because we have the root access.
&nbsp;

## 16. Disable directory listing the wordpress directory.
We will configure apache `.htaccess` to mitigate this.

```sh
sudo vim /var/www/wordpress/.htaccess

# Append the config below

Options -Indexes

# Make the .htaccess readonly.
sudo chmod 0444 /var/www/wordpress/.htaccess

sudo systemctl restart apache2
```
&nbsp;

## 17. Password Protect the "wp-login.php" page using apache.
For this the "apache2-utils" package is required which we should've installed during the wordpress setup.

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

## 18. Disable php code execution by denying php file upload.
We will allow php code execution of required php files of wordpress and disallow every other php file that might be uploaded using any file upload vulnerability. To do this we are going to disallow php file upload.

```sh
sudo vim /var/www/wordpress/wp-content/uploads/.htaccess

# append the configs

<Files *.php>
deny from all
</Files>

# Make the .htaccess readonly.
sudo chmod 0444 /var/www/wordpress/wp-content/uploads/.htaccess

sudo systemctl restart apache2
```
&nbsp;

## Conclusion
We can setup authentication for any other wordpress page using ".htaccess".
Now create a backup/clone of this entire site which will basically take a snapshot of everything including the databases and configs.
