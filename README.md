# Let's Encrypt

This is a guide for how to issue SSL certificates on Ubuntu 16.10.

## Issue Certificates

1. Ensure ports `80` and `443` are open. `letsencrypt` client will create a temporary Web server to validate the server.

2. Install `letsencrypt` client.

        apt-get udpate -y && apt-get install -y letsencrypt

3. Issue the certificates for one or more domains.
 
        certbot certonly --standalone -d pixerapp.com -d www.pixerapp.com -d cdn.pixerapp.com -d es.pixerapp.com -d api.pixerapp.com

All `letsencrypt` files are stored at `/etc/letsencrypt`.

## Generate a Certificate Chain

The `/etc/letsencrypt/archive/pixerapp.com/fullchain1.pem` file includes the full chain of certificates except the private key used to generate the certificates. Prepend the private key to have a valid certificate chain that can be used by HAProxy server.

    cd /etc/letsencrypt/archive/pixerapp.com/
    cat privkey1.pem fullchain1.pem > pixerapp.com.pem

Now move this final `pixerapp.com.pem` to a location that is used by HAProxy.
Check https://github.com/pixerapp/haproxy for how to setup HAProxy.
