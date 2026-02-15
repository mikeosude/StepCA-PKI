**Certificate Requests (Sanitized)**

**Prereqs**
- Step CA URL: `<CA_URL>`
- Root CA file: `root_ca.crt`

**Bootstrap Step CLI**
```
step ca bootstrap --ca-url <CA_URL> --root root_ca.crt
```

**Generic Request (JWK)**
```
step ca certificate "<HOSTNAME>" <SERVICE>.crt <SERVICE>.key \
  --provisioner admin-jwk
```

**ACME Directory URL**
```
<CA_URL>/acme/acme/directory
```

**Caddy**
Use `github/caddy/Caddyfile.sample`.

**Nginx**
Use `github/nginx/nginx.conf.sample` and `github/nginx/acme-request.sh.sample`.

**Apache httpd**
Use `github/httpd/httpd.conf.sample` and `github/httpd/acme-request.sh.sample`.

**Certbot**
Use `github/certbot/README.md`.

**Proxmox VE**
Configure ACME with `<CA_URL>/acme/acme/directory`.

**vCenter**
```
step ca certificate "<VCENTER_HOST>" vcenter.crt vcenter.key \
  --provisioner admin-jwk
```

**Linux OS**
```
step ca certificate "<HOSTNAME>" /etc/ssl/<SERVICE>.crt /etc/ssl/<SERVICE>.key \
  --provisioner admin-jwk
chmod 600 /etc/ssl/<SERVICE>.key
```

**Grafana**
```
step ca certificate "<GRAFANA_HOST>" /etc/grafana/grafana.crt /etc/grafana/grafana.key \
  --provisioner admin-jwk
```

**Loki**
```
step ca certificate "<LOKI_HOST>" /etc/loki/loki.crt /etc/loki/loki.key \
  --provisioner admin-jwk
```

**VictoriaMetrics**
```
step ca certificate "<VM_HOST>" /etc/victoriametrics/vm.crt /etc/victoriametrics/vm.key \
  --provisioner admin-jwk
```

**Kubernetes cert-manager**
Use `github/cert-manager/clusterissuer.yaml.sample`.

**Step CA Agent**
Use `github/stepca-agent/README.md`.

**HashiCorp Vault**
Use `github/vault/README.md`.

**SSH (optional)**
```
step ssh certificate "<USER>" ~/.ssh/id_ed25519.pub \
  --provisioner ssh
```

**OIDC (optional)**
```
step ca certificate "<USER>" user.crt user.key \
  --provisioner oidc
```
