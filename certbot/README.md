**Certbot (ACME) Samples**

**Use the Step CA ACME directory**
```
<CA_URL>/acme/acme/directory
```

**Standalone**
```
certbot certonly --standalone \
  --server <CA_URL>/acme/acme/directory \
  -d <HOSTNAME>
```

**Webroot**
```
certbot certonly --webroot \
  -w /var/www/html \
  --server <CA_URL>/acme/acme/directory \
  -d <HOSTNAME>
```
