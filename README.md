# Let's Encrypt

This is a guide for how to issue SSL certificates on Ubuntu 16.10.

Source: https://certbot.eff.org/#ubuntutyakkety-haproxy

## Issue Certificates

1. Ensure ports `80` and `443` are open. `letsencrypt` client will create a temporary Web server to validate the host 
and domain.

2. Install `letsencrypt` client.

        apt-get udpate -y && apt-get install -y letsencrypt

3. Issue the certificates for one or more domains.
 
        certbot certonly --standalone -d pixerapp.com -d www.pixerapp.com -d cdn.pixerapp.com -d es.pixerapp.com -d api.pixerapp.com

All `letsencrypt` files are stored at `/etc/letsencrypt`.

## Generate a Certificate Chain

The `/etc/letsencrypt/archive/pixerapp.com/fullchain1.pem` file includes the full chain of certificates except 
the private key used to generate the certificates. Prepend the private key to have a valid certificate chain 
that can be used by HAProxy server.

    cd /etc/letsencrypt/archive/pixerapp.com/
    cat privkey1.pem fullchain1.pem > pixerapp.com.pem

Now move this final `pixerapp.com.pem` to a location that is used by HAProxy.
Check https://github.com/pixerapp/haproxy for how to setup HAProxy.

## Renew Certificates

### Point remote domains back to the main IP

In order to renew the certificates, ensure all domains in the chain point to the main IP assigned to the host where
`certbot` script is installed. For instance, if the CDN domain `cdn.pixerapp.com` usually points to an external CDN 
service. We need to temporarily point the domain to our main host.

1. Copy and save somewhere the original destination of the remote CDN service.
2. Temporarily point the `cdn.pixerapp.com` to the main IP `pixerapp.com`.
3. Perform the same replacement operation to all other CDN like domains we have.

### Stop the proxy server

At this moment we have HAProxy bound to ports `80` (HTTP) and `443` (HTTPS). The `certbot` needs to be able to bind
to these ports, so we need to temporarily stop our proxy server.

    docker stop es_haproxy_1

Then, dry run the command to ensure the `certbot` can potentially renew all the certificates.

    # certbot renew --dry-run
    Saving debug log to /var/log/letsencrypt/letsencrypt.log
   
    -------------------------------------------------------------------------------
    Processing /etc/letsencrypt/renewal/pixerapp.com.conf
    -------------------------------------------------------------------------------
    Cert is due for renewal, auto-renewing...
    Renewing an existing certificate
    Performing the following challenges:
    tls-sni-01 challenge for pixerapp.com
    tls-sni-01 challenge for api.pixerapp.com
    tls-sni-01 challenge for cdn.pixerapp.com
    tls-sni-01 challenge for es.pixerapp.com
    tls-sni-01 challenge for www.pixerapp.com
    Waiting for verification...
    Cleaning up challenges
    
    -------------------------------------------------------------------------------
    new certificate deployed without reload, fullchain is
    /etc/letsencrypt/live/pixerapp.com/fullchain.pem
    -------------------------------------------------------------------------------
    ** DRY RUN: simulating 'certbot renew' close to cert expiry
    **          (The test certificates below have not been saved.)
    
    Congratulations, all renewals succeeded. The following certs have been renewed:
      /etc/letsencrypt/live/pixerapp.com/fullchain.pem (success)
    ** DRY RUN: simulating 'certbot renew' close to cert expiry
    **          (The test certificates above have not been saved.)

Once the dry run succeeds, we can run it for real

    certbot renew

### Generate the certificate chain

Refer to the **Generate a Certificate Chain** section above to append the private key to the generate certificates.

### Start the proxy server

Now it is time to start HAProxy.
