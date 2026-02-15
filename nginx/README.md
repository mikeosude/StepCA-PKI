**Nginx Samples**

**Files**
- `github/nginx/nginx.conf.sample`
- `github/nginx/acme-request.sh.sample`

**ACME Request Script**
- Edit placeholders in `acme-request.sh.sample`.
- Run it to request a cert and store it in `/etc/ssl`.

**Virtual Host Update**
Update your existing server block:
```
server {
  listen 443 ssl;
  server_name <HOSTNAME>;

  ssl_certificate     /etc/ssl/<HOSTNAME>.crt;
  ssl_certificate_key /etc/ssl/<HOSTNAME>.key;

  location / {
    proxy_pass http://127.0.0.1:8080;
  }
}
```

**Reload**
```
systemctl reload nginx
```
