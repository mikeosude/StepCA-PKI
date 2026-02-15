**Step CA Agent (step-agent)**

Step CA includes `step-agent` for automated certificate provisioning and renewal on hosts.

**Install**
- Install `step-cli` and `step-ca` packages on the host.

**Bootstrap**
```
step ca bootstrap --ca-url <CA_URL> --root root_ca.crt
```

**Initialize Agent**
```
step-agent init --ca-url <CA_URL> --root root_ca.crt \
  --provisioner admin-jwk
```

**Run Agent**
```
step-agent run
```

**Notes**
- Configure as a systemd service for always-on renewal.
- Use a host-specific provisioner where possible.
