**Renewal Automation Samples**

**Files**
- `github/automation/step-renew.service.sample`
- `github/automation/step-renew.timer.sample`

**Usage (systemd timer)**
- Copy to `/etc/systemd/system/`.
- Replace placeholders in the service file.
- Enable the timer.
```
systemctl daemon-reload
systemctl enable --now step-renew.timer
```

**Cron Alternative**
Run the ACME request script nightly and reload the service.
```
0 2 * * * /usr/local/bin/acme-request.sh >/var/log/acme-renew.log 2>&1
```
- Place your script in `/usr/local/bin/acme-request.sh`.
- Ensure it reloads the target service (nginx/httpd) on success.
