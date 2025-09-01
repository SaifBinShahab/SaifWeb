+++
title = "Installing MariaDB Properly on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "DevOps", "Server"]
+++

```sh

sudo apt install -y mariadb-server

sudo systemctl status mariadb # It should be active and running.

sudo mysql_secure_installation

# login as database root user.
sudo mariadb -u root -p

CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';

# Create a database named "db".
CREATE DATABASE IF NOT EXISTS db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

SHOW DATABASES;

GRANT ALL PRIVILEGES ON db.* TO 'user'@'localhost';

FLUSH PRIVILEGES;

quit;
```
