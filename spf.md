# Setting up SPF entry

SPF record tells which IPs we will be sending emails from. We specify the authorized IPs in a text DNS entry.

Add DNS entry for a `TXT` record for `@` (root) domain with content:

```
"v=spf1 a mx ip4:<ROOT_IP>.<ROOT_IP>.<ROOT_IP>.<ROOT_IP> ip4:<SMTP_IP>.<SMTP_IP>.<SMTP_IP>.<SMTP_IP> ~all"
```
