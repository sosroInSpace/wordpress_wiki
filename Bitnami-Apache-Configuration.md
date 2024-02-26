Below is a list of useful things that can be done to configure Apache on Bitnami.

**NOTE:** Changes will only take effect after restarting Apache:

`sudo /opt/bitnami/ctlscript.sh restart apache`

### Disable Bitnami Banner and Bitnami Index Page.

On line 555 in `/opt/bitnami/apache2/conf/httpd.conf`:

`Include "/opt/bitnami/apps/bitnami/banner/conf/banner.conf"`

Comment out with `#`:

`#Include "/opt/bitnami/apps/bitnami/banner/conf/banner.conf"`

### Redirect different URL to main URL (Eg. `okmg.agency` to `okmg.com`).

Create file `/opt/bitnami/apache2/conf/okmg-redirection-rules.conf` and add the following for each domain:

```
<VirtualHost *:80>
  ServerName okmg.agency
  RedirectPermanent / https://okmg.com/
</VirtualHost>

<VirtualHost *:443>
  ServerName okmg.agency
  RedirectPermanent / https://okmg.com/
  SSLEngine on
  SSLCertificateFile "/opt/bitnami/apache2/conf/okmg.com.crt"
  SSLCertificateKeyFile "/opt/bitnami/apache2/conf/okmg.com.key"
</VirtualHost>
```

Add the following to `/opt/bitnami/apache2/conf/httpd.conf` at the very bottom:

```
# OKMG Domain redirect.
Include "/opt/bitnami/apache2/conf/okmg-domain-redirect.conf"
```



