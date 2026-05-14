+++
title = "Setting up ModSecurity WAF for Nginx Webserver"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
## Note we need to perform all of these on the "server" that we're installing ModSecurity for.
&ensp;

## Install and check the nginx version
```sh
sudo apt install -y nginx

# Check the version

nginx -v
```

It should give similar results like the following:
```sh
ubnt@ubnt:~$ nginx -v
nginx version: nginx/1.24.0 (Ubuntu)
```

We can see it's version is `1.24.0`. We need to keep this in mind because we're going to build `ModSecurity` for this version of nginx.
&ensp;

## Install build tools
We need to install some build tools to build ModSecurity from source.

```sh
sudo apt install -y bison build-essential ca-certificates curl dh-autoreconf doxygen flex gawk git iputils-ping libcurl4-gnutls-dev libexpat1-dev libgeoip-dev liblmdb-dev libpcre3-dev libpcrecpp0v5 libssl-dev libtool libxml2 libxml2-dev libyajl-dev locales lua5.3 liblua5.3-dev pkg-config wget zlib1g zlib1g-dev libgd-dev

sudo snap install libxslt
```
&ensp;

## Build ModSecurity

- Clone the ModSecurity Github repository in `/opt` directory
```sh
cd /opt

sudo git clone https://github.com/owasp-modsecurity/ModSecurity
```
&ensp;

- `cd` into `ModSecurity`

```sh
cd ModSecurity
```
&ensp;

- Run the `./build` script
```sh
sudo ./build
```
&ensp;

- Run the `./configure` file, which is responsible for getting all the dependencies for the build process:
```sh
sudo ./configure
```
&ensp;

- Run the `make` command to build ModSecurity:
```sh
sudo make
```
&ensp;

- After the build process is complete, install ModSecurity by running the following command:
```sh
sudo make install
```
&ensp;

### Downloading ModSecurity-Nginx Connector
Before compiling the ModSecurity module, clone the Nginx-connector from the `/opt`directory:
```sh
cd /opt

sudo git clone --depth 1 https://github.com/owasp-modsecurity/ModSecurity-nginx.git
```
&ensp;

### Building the ModSecurity Module For Nginx

You can now build the ModSecurity module from a downloaded copy of your Nginx version by following the steps outlined below:
