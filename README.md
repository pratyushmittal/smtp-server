# Setting up postfix server

Documenting best guides for setting up postfix server.

1. Setup forward and reverse DNS: [DNS Guide](https://github.com/pratyushmittal/smtp-server/blob/master/dns-setup.md)
2. Install Postfix: [Postfix Guide](https://github.com/pratyushmittal/smtp-server/blob/master/setting-up-postfix.md)
3. Setting up SPF and DKIM: [PepiPost Guide](https://netcorecloud.com/tutorials/setup-spf-and-dkim-with-postfix-on-ubuntu/)
4. Install [pflogsumm](https://github.com/pratyushmittal/smtp-server/blob/master/pflogsumm-setup.md)
4. Setup DMARC: Add TXT record for `_dmarc.from-address.com` with value `v=DMARC1; p=quarantine; rua=mailto:postmaster@example.com; pct=100`. [https://www.uriports.com/dmarc](URIPorts) offers DMARC rua as a service for $1/month.
4. Configure deferred emails: [JRS Blog](https://jrs-s.net/2013/04/17/configuring-retry-duration-in-postfix/)
5. Setup different policies for different domains: [Policy Rate Limiting](https://web.archive.org/web/20210122150840/http://steam.io/2013/04/01/postfix-rate-limiting/). Ensure to edit both `/etc/postfix/main.cf` as well as `/etc/postfix/master.cf`
6. Disable ipv6 protocol: [Fix Postfix network unreachable](https://www.nucleiotechnologies.com/how-to-fix-postfix-smtp-network-is-unreachable-error/)
7. A good guide on line-by-line postfix config: https://jan.wildeboer.net/2022/08/Email-1-Postfix-2022/
8. Add the new server to [Google Postmaster](https://postmaster.google.com/u/0/managedomains) and [Outlook SNDS](https://sendersupport.olc.protection.outlook.com/snds/index.aspx).
9. Enroll for [Yahoo's CFL](https://senders.yahooinc.com/complaint-feedback-loop/).
10. [Monitor the logs](./troubleshooting-emails.md) for next few days.
11. Enable encryption for Gmail: [Encryption](https://serverfault.com/questions/853355/postfix-gmail-encryption)
12. [Increase SWAP](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04) to `pflogsum`. We should have memory equal to the size of `/var/log/mail.log`.


## Commands

- `systemctl reload postfix` to reload changes after modifying `main.cf`


## Email template

We often need to drop a request to AWS or other hosting provider to increase the sending limits. This template can be used for such emails.

```
(your website name) provides (what you do).
We use emails to (why?).

**Protection:**
- All emails follow with SPF and DKIM specifications
- All emails contain "List-Unsubscribe" headers for one-click unsubscribe
- Enabled SASL and firewall to send emails only from internal IPs.
- Have registered on Google Postmaster and Outlook's SNDS to track IP reputation.
- All complaints and bounces are immediately dealth with.

**Quality:**
- We email only registered users who ask for email updates.
- We have not purchased any email list.
- [any other thing you can show for quality. like reviews or age of website].
```

## TLS Setup

We need to configure TLS support on Postfix to use it with Django and Ruby libraries.

- Install [Certbot](https://certbot.eff.org/instructions?ws=other&os=ubuntufocal)
- Use `--standalone` option to create certificates by spinning up a certbot www server

The above will show something like:

```
Certificate is saved at: /etc/letsencrypt/live/post.screener.in/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/post.screener.in/privkey.pem
```

Add it to postix config

```
sudo vi /etc/postfix/main.cf

# add this
smtpd_tls_security_level = may
smtpd_tls_chain_files =
    /etc/letsencrypt/live/post.screener.in/privkey.pem,
    /etc/letsencrypt/live/post.screener.in/fullchain.pem
    
# reload postfix
systemctl reload postfix
```

Add cron to reload ssl certificates in postfix: `sudo crontab -e -uroot`

```
# reload postfix configuration every month
# the ssl renewal is done automatically every 60 days using certbot
# we can see that using `systemctl list-timers`
0 1 1 1 1 systemctl reload postfix
```
