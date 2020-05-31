# Configure firewall on server

```bash
# enable firewall
sudo ufw enable

# allow ssh
sudo ufw limit ssh

# allow firewall to send mails from particular IPs.
sudo ufw allow from <IP.IP.IP.IP> to any port 25
```