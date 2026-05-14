+++
title = "Setting up ModSecurity WAF for Apache Webserver"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Homelab", "DevOps", "Security", "Server"]
+++
## Install ModSecurity
```sh
sudo apt install -y libapache2-mod-security2
```
&nbsp;

## Setup Core Rule Set
We can learn more on [Core Rule Set Website](https://coreruleset.org/) and [Core Rule Set Github](https://github.com/coreruleset/coreruleset)

### First backup the existing ruleset

```sh
sudo mv /usr/share/modsecurity-crs /usr/share/modsecurity-crs-backup
```
&nbsp;

### Download or clone the source and signature

- To Download:
```sh
wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v4.5.0.tar.gz
wget https://github.com/coreruleset/coreruleset/releases/download/v4.5.0/coreruleset-4.5.0.tar.gz.asc
```

- To clone:
```sh
git clone https://github.com/coreruleset/coreruleset.git
```
&nbsp;

### Verify Signatures

- Import GPG Keys:
To retrieve the CRS project’s public key from public key servers using `gpg`, execute: `gpg --keyserver pgp.mit.edu --recv 0x38EEACA1AB8A6E72`(this ID should be equal to the last sixteen hex characters in the fingerprint).

It is also possible to use `gpg --fetch-key https://coreruleset.org/security.asc` to retrieve the key directly.
&nbsp;

- To verify the integrity of the release:
```sh
gpg --verify coreruleset-4.5.0.tar.gz.asc v4.5.0.tar.gz
```

This should return:

```sh
gpg: Signature made Wed Jun 30 10:05:48 2021 -03
gpg:                using RSA key 36006F0E0BA167832158821138EEACA1AB8A6E72
gpg: Good signature from "OWASP CRS <security@coreruleset.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 3600 6F0E 0BA1 6783 2158  8211 38EE ACA1 AB8A 6E72
```

If the signature was good then the verification succeeds. If a warning is displayed, like the above, it means the CRS project’s public key is known but is not trusted.
&nbsp;

- To trust the CRS project’s public key:
```sh
gpg --edit-key 36006F0E0BA167832158821138EEACA1AB8A6E72

gpg> trust
Your decision: 5 (ultimate trust)
Are you sure: Yes
gpg> quit
```
&nbsp;

- The result when verifying a release will then look like so:
```sh
gpg --verify coreruleset-4.5.0.tar.gz.asc v4.5.0.tar.gz

gpg: Signature made Wed Jun 30 15:05:48 2021 CEST
gpg:                using RSA key 36006F0E0BA167832158821138EEACA1AB8A6E72
gpg: Good signature from "OWASP CRS <security@coreruleset.org>" [ultimate]
```
&nbsp;

### Extract Core Rule Set

```sh
sudo tar -xzvf v4.5.0.tar.gz --strip-components 1 -C /usr/share/modsecurity-crs
```
&nbsp;

### Activating the CRS
```sh
sudo mkdir /usr/share/modsecurity-crs

sudo mv /usr/share/modsecurity-crs/crs-setup.conf.example /usr/share/modsecurity-crs/crs-setup.conf
```
***Note: We could also setup the crs in `/etc/crs` instead of `/usr/share/modsecurity-crs`.***
&nbsp;

#### (Optional)
In addition to crs-setup.conf.example, there are two other “.example” files within the CRS repository. These are:

```sh
rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example
rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example
```

These files are designed to provide the rule maintainer with the ability to modify rules (see false positives and tuning) without breaking forward compatibility with rule set updates. These two files should be renamed by removing the .example suffix. **This will mean that installing updates will *not* overwrite custom rule exclusions**. To rename the files in Linux, use a command similar to the following:

```sh
sudo mv rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
sudo mv rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```
&nbsp;

## Configuring and Setting Up ModSecurity

### Activate the config
```sh
sudo mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
```
&nbsp:

### Configure Apache

```sh
sudo vim /etc/apache2/apache2.conf

# We can put these configs the very beginning of the config. You can try a different place if it causes any isues.

<IfModule security2_module>
	Include /etc/modsecurity/modsecurity.conf
	Include /usr/share/modsecurity-crs/crs-setup.conf
	Include /usr/share/modsecurity-crs/rules/*.conf
</IfModule>
```
&nbsp;

Also it's a good practice to include similar configs in the apache default sites-enabled config file.

```sh
sudo vim /etc/apache2/sites-enabled/000-default.conf

# Now put the configs here. We can put the configs After "DocumentRoot" Inside "<VirtualHost *:80>".

# Note: In this config we didn't include "/etc/modsecurity/modsecurity.conf" because in our testing it was proving to be a misconfiguration.

<IfModule security2_module>
	Include /usr/share/modsecurity-crs/crs-setup.conf
	Include /usr/share/modsecurity-crs/rules/*.conf
</IfModule>
```
&nbsp;

### Activate ModSecurity

```sh
sudo vim /etc/modsecurity/modsecurity.conf

# Change the config from "SecRuleEngine DetectionOnly" to "SecRuleEngine On"

SecRuleEngine On
```
&nbsp;

## Now Restart Apache
```sh
sudo systemctl restart apache2
```
&nbsp;

## Now we can test the security. (Recommended)

Always verify that CRS is installed correctly by sending a ‘malicious’ request to your site or application, for instance:

```sh
curl 'https://www.example.com/?foo=/etc/passwd&bar=/bin/sh'
```

Depending on your configurated thresholds, this should be detected as a malicious request. If you use blocking mode, you should receive an Error 403. The request should also be logged to the audit log, which is usually in `/var/log/modsec_audit.log`.
&nbsp;

Done!