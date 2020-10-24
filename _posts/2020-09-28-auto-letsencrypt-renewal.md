---
layout: post
title: Automatic SSL renewal and deployment using LetsEncrypt SSL certificates
date: 2020-09-28 14:19
category: SSL
author: Zeb Rasco
tags: [ssl, vmware, letsencrypt]
summary: 
---

When you set up a lab, you'll quickly realize that the web services involved will use self-signed certificates. This results in ugly browser errors such as the one below:

[![](/assets/2020-09-28-auto-letsencrypt-renewal/2020-09-28-14-23-19.png)](/assets/2020-09-28-auto-letsencrypt-renewal/2020-09-28-14-23-19.png)

Obviously we don't want to have this kind of setup in a real environment, but even in a lab it seems unprofessional and sloppy. So I decided to fix it.

# Setting up SSL certificates from Let's Encrypt

This is where [Let's Encrypt](http://www.letsencrypt.org) comes into play. Once you purchase a domain from a registrar, you're able to purchase an SSL certificate from that registrar for an additional fee, typically one for each website. But Let's Encrypt is free! So let's use that instead.

Typically, we would set up certbot and use that to download the SSL certificates. However, in my case, I have a COX residential account and port 80 is blocked. Thus I was unable to use a normal HTTP challenge to authenticate my domain, zebslab.org. So I found out about another ACMEv2 client called dehydrated and decided to use that for a DNS-based challenge.

## Set up dehydrated & domains.txt

All of these steps work in Ubuntu Server 18.04

First, clone the dehydrated repo to your home directory.

```bash
cd ~
git clone https://github.com/dehydrated-io/dehydrated.git
cd dehydrated
```

Now, create a new account to use with ACMEv2. It doesn't really matter how many new accounts you create.

```bash
./dehydrated --register --accept-terms
```

Finally, create a new file called ~/dehydrated/domains.txt which will contain your domain and subdomains. In my case, I decided to use a wildcard and a specific cert for my vCenter Appliance.

Contents of domain.txt:
```txt
zlvca.zebslab.org > zlvca_zebslab_org
*.zebslab.org > zebslab_org
```

I put the wildcard last to avoid problems. The text to the left is the exact subdomain, and the text to the right is an alias to that domain. Each line item will correspond to a different set of SSL certificates, and the alias will be the directory name containing them. Note that each subdomain will have to have a corresponding CNAME or A record in your DNS settings for the domain (except the wildcard, obviously).

Now, we can move onto setting up GoDaddy.

## Set up GoDaddy

#### API key/secret
In order to programmatically update GoDaddy DNS records, you'll need an API key and secret. You can find the page easily with a google search. At the time of this writing, you can find the [page here.](https://developer.godaddy.com/getstarted)

#### TXT records
You will need to create a TXT record for each subdomain and a generic TXT record if you choose to use a wildcard. The format for the name of the TXT records are:

```
Wildcard: _acme-challenge
Subdomain: _acme-challenge.<subdomain name>
```

So in my case, I created two TXT records: _acme-challenge and _acme-challenge.zlvca

The TXT records can be set to any value, as they'll be overwritten during the certificate renewal process. With the TXT records in place we're ready to glue everything together.

## Putting it all together
#### Set up GoDaddy hook in dehydrated

A hook is simply a shell script that dehydrated executes during certain events in the certificate generation process. The hook script file will contain various functions, which represent the callbacks for those events.


Create a new directory structure to hold the hook file, ~/dehydrated/hooks/godaddy/ and set the script permission to executable:

```bash
mkdir -p ~/dehydrated/hooks/godaddy/
cd ~/dehydrated/hooks/godaddy/
wget http://zrasco.github.io/assets/2020-09-28-auto-letsencrypt-renewal/hook.sh
chmod +x hook.sh
```
[Download this hook file](/assets/2020-09-28-auto-letsencrypt-renewal/hook.sh) 

Don't forget to set executable permissions on the hook file using the last line above.

#### Set up the renewal script
Finally, we'll set up the script to make everything work. I like to run this as a cronjob once a month, which sends me a nice email report of the results.

```bash
#!/bin/bash

# Godaddy API Key/Secret needed for dehydrated hook
export GODADDY_KEY='GODADDY API KEY HERE'
export GODADDY_SECRET='GODADDY API SECRET HERE'

/home/zrasco/dehydrated/dehydrated -c --force -t dns-01 -k '/home/zrasco/dehydrated/hooks/godaddy/hook.sh' --out /etc/letsencrypt/live/

# Copy the certs to the webserver. Various cronjobs and scheduled tasks on other machines will pick these up later
WEBROOT="/var/www/html/zllamp.zebslab.org/public_html"

# Wildcard cert
CERTDIR="/etc/letsencrypt/live/zebslab_org"
cp -H $CERTDIR/privkey.pem $WEBROOT/wildcardCert/
cp -H $CERTDIR/cert.pem $WEBROOT/wildcardCert/
cp -H $CERTDIR/chain.pem $WEBROOT/wildcardCert/
cp -H $CERTDIR/fullchain.pem $WEBROOT/wildcardCert/

# vCenter cert
CERTDIR="/etc/letsencrypt/live/zlvca_zebslab_org"
cp -H $CERTDIR/privkey.pem $WEBROOT/zlvcaCert/
cp -H $CERTDIR/cert.pem $WEBROOT/zlvcaCert/
cp -H $CERTDIR/chain.pem $WEBROOT/zlvcaCert/
cp -H $CERTDIR/fullchain.pem $WEBROOT/zlvcaCert/

# Set permissions
chown -R www-data:www-data $WEBROOT/

# Reload apache
systemctl reload apache2

# Run post-renewal scripts below
```

The script above is ran as root from the cron invocation, so it's able to copy the certs to the default global directory of /etc/letsencrypt/live. It also copies the newly-generated certs to the apache web root (to which access is carefully controlled via a whitelist, because the private key is available).

In addition, I have a few other scripts which update the ESXI and vCenter Appliance VM, but have omitted those for the sake of simplicity. I will cover these topics in future blog posts.

That's it! Hope you enjoyed and please leave a comment if you have any questions.

Zeb