**Step CA PKI Blueprint**

**Purpose**
A public-ready guide for building a PKI with Step CA.

**Scope**
- Create a 50-year root CA with OpenSSL.
- Create and sign a Step CA intermediate.
- Configure Step CA with MySQL (or your chosen DB).
- Enable ACME for automated renewals.
- Provide automation examples for renewal.
- Configure Telegraf to ship metrics to VictoriaMetrics and logs to Loki.

**Conventions**
Replace all placeholders in angle brackets with your values.
- `<CA_HOST>`
- `<CA_URL>`
- `<DB_HOST>` `<DB_NAME>` `<DB_USER>` `<DB_PASSWORD>`
- `<ROOT_CN>` `<INTERMEDIATE_CN>`
- `<ORG>` `<OU>` `<CITY>` `<STATE>` `<COUNTRY>`

**Table of Contents**
1. Generate a 50-Year Root CA (OpenSSL)
2. Generate an Intermediate for Step CA (OpenSSL)
3. Install Step CA
4. Initialize Step CA with the Existing Root/Intermediate
5. Configure Step CA (MySQL + HTTP)
6. Enable ACME (for Auto-Renewals)
7. Auto-Renewals (Examples)
8. Telegraf Monitoring + Log Shipping
9. Publishing Notes

---

**Repo Layout**
- Step CA samples: `github/stepca/README.md`
- Step CA config samples: `github/stepca/ca.json.sample`
- Step CA systemd service: `github/stepca/step-ca.service.sample`
- Step CA environment file: `github/stepca/step-ca.env.sample`
- Telegraf samples: `github/telegraf/README.md`
- Telegraf main config: `github/telegraf/telegraf.conf.sample`
- Telegraf metrics inputs: `github/telegraf/telegraf.d/system-metrics.conf.sample`
- Telegraf logs inputs: `github/telegraf/telegraf.d/logs.conf.sample`
- Renewal automation: `github/automation/README.md`
- cert-manager: `github/cert-manager/README.md`
- Caddy: `github/caddy/README.md`
- Nginx: `github/nginx/README.md`
- Apache httpd: `github/httpd/README.md`
- Certificate request guide: `github/request.md`
- Proxmox: `github/proxmox/README.md`
- vCenter: `github/vcenter/README.md`
- Certbot: `github/certbot/README.md`
- Step CA Agent: `github/stepca-agent/README.md`
- HashiCorp Vault: `github/vault/README.md`

**1) Generate a 50-Year Root CA (OpenSSL)**
```
mkdir -p pki/root
cd pki/root

cat > root-openssl.cnf <<'CONF'
[ req ]
default_bits       = 4096
default_md         = sha256
prompt             = no
distinguished_name = dn
x509_extensions    = v3_ca

[ dn ]
C  = <COUNTRY>
ST = <STATE>
L  = <CITY>
O  = <ORG>
OU = <OU>
CN = <ROOT_CN>

[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, keyCertSign, cRLSign
CONF

# Root key
openssl genrsa -out root.key 4096

# Root cert (50 years = 18250 days)
openssl req -x509 -new -nodes \
  -key root.key \
  -sha256 -days 18250 \
  -out root.crt \
  -config root-openssl.cnf
```

**2) Generate an Intermediate for Step CA (OpenSSL)**
```
mkdir -p ../intermediate
cd ../intermediate

cat > int-openssl.cnf <<'CONF'
[ req ]
default_bits       = 4096
default_md         = sha256
prompt             = no
distinguished_name = dn

[ dn ]
C  = <COUNTRY>
ST = <STATE>
L  = <CITY>
O  = <ORG>
OU = <OU>
CN = <INTERMEDIATE_CN>
CONF

# Intermediate key
openssl genrsa -out intermediate.key 4096

# CSR
openssl req -new -key intermediate.key \
  -out intermediate.csr \
  -config int-openssl.cnf

# Sign intermediate with root (10 years example = 3650 days)
openssl x509 -req \
  -in intermediate.csr \
  -CA ../root/root.crt \
  -CAkey ../root/root.key \
  -CAcreateserial \
  -out intermediate.crt \
  -days 3650 -sha256
```

**3) Install Step CA**
Install `step-ca` and `step-cli` using your OS package manager or the official Smallstep repo.

**4) Initialize Step CA with the Existing Root/Intermediate**
```
export STEPPATH=/etc/step-ca
mkdir -p /etc/step-ca/certs /etc/step-ca/secrets

# Copy PKI material
install -m 0644 pki/root/root.crt /etc/step-ca/certs/root_ca.crt
install -m 0644 pki/intermediate/intermediate.crt /etc/step-ca/certs/intermediate_ca.crt
install -m 0600 pki/intermediate/intermediate.key /etc/step-ca/secrets/intermediate_ca_key

# Create password file for intermediate key (if encrypted)
install -m 0600 /dev/null /etc/step-ca/password.txt
# Add your passphrase

# Initialize config
step ca init \
  --name "<CA_NAME>" \
  --dns <CA_HOST> \
  --address ":443" \
  --root /etc/step-ca/certs/root_ca.crt \
  --key /etc/step-ca/secrets/intermediate_ca_key \
  --provisioner "admin-jwk" \
  --password-file /etc/step-ca/password.txt
```

**5) Configure Step CA (MySQL + HTTP)**
Edit `/etc/step-ca/config/ca.json`:
```
"address": ":443",
"insecureAddress": "0.0.0.0:80",
"root": "/etc/step-ca/certs/root_ca.crt",
"crt": "/etc/step-ca/certs/intermediate_ca.crt",
"key": "/etc/step-ca/secrets/intermediate_ca_key",
"db": {
  "type": "mysql",
  "dataSource": "<DB_USER>:<DB_PASSWORD>@tcp(<DB_HOST>:3306)/<DB_NAME>?parseTime=true"
}
```

**6) Enable ACME (for Auto-Renewals)**
```
step ca provisioner add acme --type=ACME \
  --ca-url <CA_URL> \
  --root /etc/step-ca/certs/root_ca.crt
```

ACME Directory URL:
```
<CA_URL>/acme/acme/directory
```

**7) Auto-Renewals (Examples)**
- Systemd timer calling `step ca renew` on a schedule.
- ACME clients (Caddy, cert-manager, certbot) using the ACME directory URL.

Example systemd unit:
```
step ca renew --force /etc/ssl/your-service.crt /etc/ssl/your-service.key
```

**8) Telegraf Monitoring + Log Shipping**
`/etc/telegraf/telegraf.conf`:
```
[global_tags]
  app = "stepCA"
  env = "prod"

[[outputs.http]]
  url         = "http://<VICTORIAMETRICS_HOST>:8428/api/v1/write"
  data_format = "prometheusremotewrite"
  content_encoding = "snappy"

[[outputs.loki]]
  domain   = "http://<LOKI_HOST>:3100"
  endpoint = "/loki/api/v1/push"
  gzip_request = true
```

`/etc/telegraf/telegraf.d/logs.conf`:
```
[[inputs.tail]]
  files = [
    "/var/log/stepca/step-ca.log",
    "/var/log/stepca/step-ca.err"
  ]
  from_beginning = false
  watch_method = "inotify"
  name_override = "logfile"
  data_format = "grok"
  grok_patterns = ["%{GREEDYDATA:message}"]
```

`/etc/telegraf/telegraf.d/system-metrics.conf`:
```
[[inputs.systemd_units]]
  pattern = "step*"
  unittype = "service"
```

---

**Publishing Notes**
- Do not commit secrets, passwords, or private keys.
- Use `.env` files or a secrets manager for credentials.
- Keep `root.key` and `intermediate.key` offline or in HSM/KMS if possible.

**Recommended Step CA Server Specs**
- OS: RHEL or AlmaLinux
- Storage: 8GB minimum
- RAM: 512MB minimum
- SELinux: enforcing on RHEL/AlmaLinux (AppArmor on Debian-based)
- AIDE: enabled where supported
- Disk encryption: recommended if the root key remains on the CA host
