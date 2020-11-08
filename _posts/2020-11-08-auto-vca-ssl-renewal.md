---
layout: post
title: Automated renewal of vCenter Appliance SSL certificates
date: 2020-11-08 11:12
category: SSL
author: Zeb Rasco
tags: [ssl, vmware, letsencrypt]
summary: 
---

In my [previous post]({{ page.previous.url }}), we reviewed the framework of my automated SSL certificate renewal process using LetsEncrypt. Next, we'll talk about how to automatically renew the SSL certificates used by the vCenter Appliance (VCA) using a series of REST API calls which are invoked from a renewal script, using cURL.

We'll make the assumption that there is a pre-existing VCA appliance. As of this writing, this procedure works with vCenter 7.

# Set up certificate trust store

Before the VCA appliance will accept certificate renewals, we need to add both the root authority certificate and Let's Encrypt intermediate CA certificate to the VCA trust store. You can get them [here](https://letsencrypt.org/certificates/)

To complete the the trust chain, I used the following certificates:

[DST Root CA X3 (Root)](https://letsencrypt.org/certs/trustid-x3-root.pem.txt)

[Let’s Encrypt Authority X3 (Intermediate)](https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.der)

Navigate to the trust store by going to the VCA client Menu->Administration. Then select Certificate Management.

[![](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-32-42.png)](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-32-42.png)

Upload both the root CA and intermediate CA certificates above in .pem format into the trust store. Once uploaded, you should be able to see them alongside the vSphere CA root certificate:

[![](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-48-53.png)](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-48-53.png)

[![](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-49-47.png)](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-49-47.png)

[![](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-50-27.png)](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-11-50-27.png)

While there is most likely a way to update certificates in the trust store using REST API calls, it's probably not worth the effort. Root and intermediate certificates have a longer expiration and don't need to be updated often.

With the trust store set up, we can now move onto updating the VCA machine certificate.

# Updating the machine certificate

NOTE: The vCenter Appliance will NOT accept a wildcard certificate. You will need to issue a separate one specifically for the machine.

The certificate can be updated manually through the administration tool or through a series of REST API calls. You will need to make 3 API calls:

1. Obtain a session token
1. Update the SSL certificate using the token
1. Close the session

We can do all 3 using a shell script and cURL. Here is this script I wrote to do this job:

```bash
#!/bin/bash

# update_vca.sh

# You will need to set the following variables before invoking this script:
#
# export CERTDIR='/etc/letsencrypt/live/zlvca_zebslab_org'
# export SERVER='https://zlvca.zebslab.org'
# export CRED="administrator@vsphere.local:$(pass ZLVCA)"
#
# $AFTER_RENEWAL_DIR/update_vca.sh

# Step 1a - Get the session ID.
SESSION_ID=$(curl -s -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' --header 'vmware-use-header-authn: test' --header 'vmware-api-session-id: null' -u $CRED "$SERVER/rest/com/vmware/cis/session" --insecure | python3 -c "import sys, json; print(json.load(sys.stdin)['value'])")
echo "Session ID: $SESSION_ID"

# Step 1b - Get both the certificate and private key into the format we need. Newlines must be converted into \n using awk or a similar command
PRIVKEY=$(awk -v ORS='\\n' '1' "$CERTDIR/privkey.pem")
CERT=$(awk -v ORS='\\n' '1' "$CERTDIR/cert.pem")

# Step 1c - Build the JSON request body. You can find this on your VCA appliance in the testing section.
REQUEST_BODY="{ \"spec\" : { \"cert\" : \"$CERT\", \"key\" : \"$PRIVKEY\"  } }"

# Step 2 - Update the certificate using the request body
echo "Updating cert..."
curl --insecure -X PUT "$SERVER/rest/vcenter/certificate-management/vcenter/tls" \
                -H "vmware-api-session-id: $SESSION_ID" \
                -H "Content-type: application/json" \
                -d "$REQUEST_BODY"

# Step 3 - Close the session
echo "Deleting session..."
curl --insecure -X DELETE "$SERVER/rest/com/vmware/cis/session" -H "vmware-api-session-id: $SESSION_ID"
echo "Done!"
```

Save this script as update_vca.sh. Don't forget to give it executable permissions:

```bash
chmod +x update_vca.sh
```

In my case, I invoke this script from my Let's Encrypt renewal script:
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
AFTER_RENEWAL_DIR=$(dirname "$(readlink -f "$0")")

# (Other scripts go here)

# Update vCenter Appliance
export CERTDIR='/etc/letsencrypt/live/zlvca_zebslab_org'
export SERVER='https://zlvca.zebslab.org'
export CRED="administrator@vsphere.local:$(pass ZLVCA)"
$AFTER_RENEWAL_DIR/update_vca.sh
```

Obviously you'll need to update the variables with your own info before you call the script. I installed the pass package and use it to store my passwords, so that I don't have to keep them in plaintext in the script.

There is no need to restart the vCenter Appliance or services, as the API call takes care of this behind the scenes. In my lab it takes anywhere from 5-10 minutes.

When the process is complete, you can verify the procedure was successful by looking at the certificate in the browser:

[![](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-12-34-57.png)](/assets/2020-11-08-auto-vca-ssl-renewal/2020-11-08-12-34-57.png)

And that's all there is to it! Give me a shout in the comments if you have any questions.

Zeb