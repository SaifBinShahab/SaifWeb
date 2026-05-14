+++
title = "Securing Nginx Webserver"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
## Nginx default Configuration File
It is located in `/etc/nginx/nginx.conf`.
&nbsp;

## Disble Server information leak
To do this we're gonna vim into  `/etc/nginx/nginx.conf`.

```sh
sudo vim /etc/nginx/nginx.conf

# And uncomment "server_tokens off"

server_tokens off

```
&nbsp;

## Disble Server Signature and Click Jacking Attack
```sh
sudo vim /etc/nginx/nginx.conf

# Add the config before "Virtual Hosts Config".

proxy_hide_header       X-Powered-By;
add_header      X-Frame_options SAMEORIGIN;

```
&nbsp;

The `proxy_hide_header` disables Server Signature and the `add_header` disables "iframing" of our website prevententing Click Jacking Attacks.
&nbsp;

## Protect webpages with password
We're gonna do this with `htpasswd` which is a part of `apache2-utils`.

```sh
sudo htpasswd -c /etc/nginx/.htpasswd username

# Then inside the "Virtual Hosts Config" section of the "/etc/nginx/nginx.conf" file or the seperate virtual hosts config file.

# For the authentication, inside "server" section we need to have the following config.
auth_basic	"Dev Team Only";
auth_basic_user_file	/etc/nginx/.htpasswd;

# And inside "location" section
auth_basic	on;
```
&nbsp;

So, the final config of the Virtual Hosts Config should look something like this. This is bare minumum sample config btw.
```sh
server {
listen	80;
server_name	localhost;
auth_basic	"Dev Team Only";
auth_basic_user_file	/etc/nginx/.htpasswd;

location /var/www/html {
		auth_basic	on;
		root /var/www/html;
	}
}
```
&nbsp;

Done!