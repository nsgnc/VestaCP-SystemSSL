# VestaCP System SSL
The target was to use Let's Encrypt for VestaCP SSL Cert based in /usr/local/vesta/ssl. First try was to do add the server domain to the admin user, create a let's encrypt certificate (v-add-letsencrypt-domain) and do a symlink with the VestaCP SSL. A symlink produces a permission problem, because the exim4 service needs root:mail ownership to read the cert files. So I've written a small script, that compares the Let's Encrypt Certificate with the system certificate, if the Let's Encrypt Cert is updated, it will copy, change permission and restart the services.

```bash
nano /etc/cron.daily/vesta_ssl
```
Please change the [USER] and [DOMAIN] part with your system domain.
```bash
#!/bin/bash

cert_src="/home/[USER]/conf/web/ssl.[DOMAIN].pem"
key_src="/home/[USER]/conf/web/ssl.[DOMAIN].key"
cert_dst="/usr/local/vesta/ssl/certificate.crt"
key_dst="/usr/local/vesta/ssl/certificate.key"

if ! cmp -s $cert_dst $cert_src
then
        # Copy Certificate
        cp $cert_src $cert_dst

        # Copy Keyfile
        cp $key_src $key_dst

        # Change Permission
        chown root:mail $cert_dst
        chown root:mail $key_dst

        # Restart Services
        service vesta restart &> /dev/null
        service exim4 restart &> /dev/null
fi
```

```bash
chmod +x /etc/cron.daily/vesta_ssl
```