**Apache httpd Samples**

**Files**
- `github/httpd/httpd.conf.sample`
- `github/httpd/acme-request.sh.sample`

**ACME Request Script**
- Edit placeholders in `acme-request.sh.sample`.
- Run it to request a cert and store it in `/etc/ssl`.

**Virtual Host Update**
Update your existing vhost:
```
<VirtualHost *:443>
  ServerName <HOSTNAME>
  SSLEngine on
  SSLCertificateFile /etc/ssl/<HOSTNAME>.crt
  SSLCertificateKeyFile /etc/ssl/<HOSTNAME>.key

  ProxyPass / http://127.0.0.1:8080/
  ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

**Reload**
```
systemctl reload httpd
```
