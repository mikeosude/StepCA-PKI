**HashiCorp Vault (PKI)**

Vault can act as a client requesting certs from Step CA or as a separate PKI. This guide shows using Step CA to issue certs for Vault TLS.

**Issue a Vault TLS cert**
```
step ca certificate "<VAULT_HOST>" vault.crt vault.key \
  --provisioner admin-jwk
```

**Use in Vault config**
```
listener "tcp" {
  address = "0.0.0.0:8200"
  tls_cert_file = "/etc/vault/vault.crt"
  tls_key_file  = "/etc/vault/vault.key"
}
```

**Notes**
- Ensure the cert SAN matches the Vault address.
- Use ACME for automated renewals if preferred.
