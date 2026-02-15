**vCenter Certificate Request**

**Generate Cert + Key with Step CLI**
```
step ca certificate "<VCENTER_HOST>" vcenter.crt vcenter.key \
  --provisioner admin-jwk
```

**Notes**
- Import the cert/key into vCenter following your standard process.
- Ensure the SAN matches the vCenter FQDN.
