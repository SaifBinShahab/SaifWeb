+++
title = "Installing Nextcloud server on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "Server"]
+++
```sh
sudo apt update && sudo apt full-upgrade -y

aria2c -j32 -s32 https://download.nextcloud.com/server/releases/latest.zip -o nextcloud.zip

sudo apt install -y mariadb-server apache2  unzip smbclient imagemagick libmagickcore-6.q16-6-extra memcached libmemcached-tools php php-fpm php-imagick php-memcached php-apcu php-gd php-mysql php-curl php-mbstring php-intl php-gmp php-xml php-zip php-bz2 php-common php-cli php-bcmath

sudo systemctl status mariadb # It should be active and running.

sudo mysql_secure_installation

sudo mariadb -u root -p

CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'password';

CREATE DATABASE IF NOT EXISTS db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

SHOW DATABASES;

GRANT ALL PRIVILEGES ON db.* TO 'nextcloud'@'localhost';

FLUSH PRIVILEGES;

quit;

sudo cp nextcloud.zip /var/www/

cd /var/www

sudo unzip nextcloud.zip

sudo chown -R www-data:www-data /var/www/nextcloud

sudo phpenmod bcmath gmp imagick intl

sudo a2dissite 000-default.conf

sudo a2enconf php8.2-fpm

sudo a2enmod headers rewrite mpm_event http2 mime proxy proxy_fcgi setenvif alias dir env ssl proxy_http proxy_wstunnel

# apache config

<VirtualHost *:80>
        Protocols h2 h2c http/1.1
        ServerName 10.11.0.23
        DocumentRoot /var/www/nextcloud

        <IfModule mod_headers.c>
          Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
        </IfModule>

        <FilesMatch \.php$>
          SetHandler "proxy:unix:/var/run/php/php8.2-fpm.sock|fcgi://localhost"
        </FilesMatch>

        <Directory /var/www/nextcloud/>
                Satisfy Any
                Require all granted
                Options FollowSymlinks MultiViews
                AllowOverride All
                <IfModule mod_dav.c>
                        Dav off
                </IfModule>
        </Directory>

        ErrorLog /var/log/apache2/nextcloud-error.log
        CustomLog /var/log/apache2/nextcloud-access.log common
</VirtualHost>


sudo a2ensite nextcloud.conf

sudo vim /etc/memcached.conf # change "-m 64" to "-m 1024"

sudo vim /etc/php/8.2/fpm/pool.d/www.conf
# config
pm.max_children = 80
pm.start_servers = 20
pm.min_spare_servers = 20
pm.max_spare_servers = 60

# uncomment these lines:
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp

### php config

sudo vim /etc/php/8.2/fpm/php.ini # And
sudo vim /etc/php/8.2/apache2/php.ini

memory_limit = 2048M
post_max_size = 1024M
upload_max_filesize = 5G
max_execution_time = 360
date.timezone = Asia/Dhaka
pdo_mysql.default_socket=/run/mysqld/mysqld.sock # If cannot find the sock file the look into "sudo systemctl status mariadb". It will be in the status log.

# down in opcache settings:
opcache.enable=1
opcache.memory_consumption=1024
opcache.interned_strings_buffer=64
opcache.max_accelerated_files=150000
opcache.max_wasted_percentage=15
opcache.revalidate_freq=60
opcache.save_comments=1
opcache.jit=1255
opcache.jit_buffer_size=256M


sudo systemctl restart apache2
sudo systemctl restart memcached
sudo systemctl restart php8.2-fpm


sudo mkdir /storage/nextcloud_data
sudo chown -R www-data:www-data /storage/nextcloud_data
sudo chmod 777 /storage/nextcloud_data

# Go to nextcloud webpage and initialize.

# nextcloud config

sudo vim /var/www/nextcloud/config/config.php

'memcache.local' => '\OC\Memcache\APCu',
'memcache.distributed' => '\OC\Memcache\Memcached',
'memcache.locking' => '\OC\Memcache\Memcached',
'default_phone_region' => 'BD',
'maintenance_window_start' => 1, # The time here is in UTC. We are setting it to 7.
'filesystem_check_changes' => 1,


*Optional but highly recommended*
Enable OCC -- nextcloud command line function

sudo vim /etc/php/8.2/mods-available/apcu.ini

add the following line at the bottom:
apc.enable_cli=1

save and exit


in nextcloud go to
administration settings  --> basic settings --> change from Ajax to Cron (recommended)

if you want to test if cron is working
sudo -u www-data php -f /var/www/nextcloud/cron.php

if you see nothing it is working, if you get an error it is not working


sudo crontab -u www-data -e

# Append the line in the configuration. This should run the command every 5 minutes. (Recommended by Nextcloud)
*/5 * * * * php -f /var/www/nextcloud/cron.php

# To make it run the command every day at 13:00 Hrs.
0 13 * * * php -f /var/www/nextcloud/cron.php

# You can verify if the cron job has been added and scheduled by executing:
sudo crontab -u www-data -l


# Install "Preview Generator" and "Memories" from Nextcloud app store and setup memories app from the dashboard.
sudo -u www-data php /var/www/nextcloud/occ memories:index

```
