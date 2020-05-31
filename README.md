# Setting up SMTP server for sending emails

This is a guide on how we can setup our own SMTP server for sending lots of emails.


## Topics

1. [Why take the pain?](./why-smtp.md) It is 100x cheaper and scalable.
2. Finding a good host
    1. [Picking a good host](./choosing-host.md)
    2. [Checking IP blacklist on common spam tracking services](./blacklist-status.md)
3. Setting up postfix
    1. [Setting up server](./server-setup.md)
    2. [Installing postfix and authentication](setting-up-postfix.md)
    3. Enable DKIM
    4. Setup SPF
4. Monitoring
    1. Check if SPF and DKIM tests are passing
    2. Monitor server health using Google Postmaster
    3. Monitoring spams using Hotmail's SNDS
    4. Script to get daily error logs
5. Whitelisting
    1. Check on common services
    2. Whitelist on Hotmail
6. [Useful commands for debugging or handling send queue](./postfix-commands.md)


## Overall Checklist
- [ ] Choose a host which provides reverse DNS and whitelisted IPs: [choosing host](./choosing-host.md)
- [ ] Start a new virtual server: use FQDN (eg `mail.domain.com`) as hostname.
- [ ]  Assign Elastic IP as they are negotiated for whitelisting.
- [ ]  Check Server IP Quality on blacklisting services: [blacklisting status](./blacklisting-status.md)

- [ ] Open a support ticket with your host to allow SMTP.
- [ ] Enable firewall: [server setup](./server-setup.md)
- [ ] Install postfix on server: [installing postfix](./setting-up-postfix.md)
- [ ] Send a test mail from SMTP server
- [ ] Send a test mail from another machine using authentication

- [ ] Fine tune installation
- [ ] Setup SPF DNS record
- [ ] Setup DKIM DNS record
- [ ] Setup PTR record (reverse DNS): done at [host level](http://joshua5201.github.io/blog/2015/06/06/setting-up-reverse-dns-ptr-record-in-digitalocean/): check status [here](https://mxtoolbox.com/SuperTool.aspx?action=ptr%3a159.65.157.119&run=toolpage#)
    - [ ]  [Floating IP on DigitalOcean does not support PTR](https://www.digitalocean.com/community/questions/how-do-i-set-the-ptr-for-a-floating-ip).
- [ ]  Send a test mail to see results: `echo "Testing" | mail -aReply-To:your.email@gmail.com -s "Testing Mail" check-auth@verifier.port25.com`

- [ ]  Setup [Google Postmaster account](https://postmaster.google.com/u/0/dashboards#do=screener.in&st=domainReputation&dr=7) to debug mail quality
- [ ]  Setup [Outlook SNDS (smart network data service)](https://sendersupport.olc.protection.outlook.com/snds/index.aspx?wa=wsignin1.0)
- [ ]  Subscribe to JMRP: [https://kb.iweb.com/hc/en-us/articles/230267648-Subscribing-to-Microsoft-JMRP-and-SNDS](https://kb.iweb.com/hc/en-us/articles/230267648-Subscribing-to-Microsoft-JMRP-and-SNDS)
