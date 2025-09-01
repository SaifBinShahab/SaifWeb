+++
title = "Install SSL/TLS Certificate on Linux by Let's Encrypt"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "DevOps", "Server"]
+++
# Installing SSL/TLS certificate
- A lot of the domain providers should provide an SSL certificate.
- Cloudflare privides a free SSL certificate upon setting it up as a dns. Also, when using Cloudflare make sure to use the reverse proxy service of Cloudflare and CDN and other domain and server security (like DDOS) features of Cloudflare.
- You can also install a free SSL certificate by "Let's Encrypt" which is a project of EFF. The procedure is as follows.

```sh
sudo apt install -y certbot python-certbot-apache python-certbot-nginx

# Install the certificate by providing the webserver technology name. We can also use "--nginx".

sudo certbot --apache

# This will detect connected domains to this server and ask to install certificates for the domains. It will allow to install certificates for multiple domains connected to the server.
```
