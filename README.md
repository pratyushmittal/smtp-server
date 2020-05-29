# Setting up SMTP server for sending emails

This is a guide on how we can setup our own SMTP server for sending lots of emails.


## Topics

1. Why take the pain? It is 100x cheaper and scalable.
2. Finding a good host
3. Setting up postfix
    3.1. Setting up server
    3.2. Installing postfix and authentication
    3.3. Enable DKIM
    3.5. Setup SPF
4. Monitoring
    4.1. Check if SPF and DKIM tests are passing
    4.2. Monitor server health using Google Postmaster
    4.3. Monitoring spams using Hotmail's SNDS
    4.4. Script to get daily error logs
5. Whitelisting
    5.1. Check on common services
    5.2. Whitelist on Hotmail
6. Useful commands


## Overall Checklist
- [ ] Have separate addresses (and servers) for sending different kind of emails.
- [ ] Choose a host which provides reverse DNS and whitelisted IPs.
- [ ] Start a new virtual server: use FQDN (eg `mail.domain.com`) as hostname.
- [ ] Open a support ticked to allow SMTP.
- [ ] Install postfix on server

- [ ]  Setup [Google Postmaster account](https://postmaster.google.com/u/0/dashboards#do=screener.in&st=domainReputation&dr=7) to debug mail quality
- [ ]  Setup [Outlook SNDS (smart network data service)](https://sendersupport.olc.protection.outlook.com/snds/index.aspx?wa=wsignin1.0)
- [ ]  Check Server IP Quality at DNSBLs or browse web from that IP
    - [ ]  Send an email to outlook user. Request for [whitelisting](https://support.microsoft.com/en-us/supportrequestform/8ad563e3-288e-2a61-8122-3ba03d6b8d75) if blocked.
    - [ ]  Check on [http://www.anti-abuse.org/multi-rbl-check/](http://www.anti-abuse.org/multi-rbl-check/?__cf_chl_jschl_tk__=95ba4dfa95c8ea03513abc7c7d6e0baeab98b706-1575954645-0-AbQsOpQH-CRqxIkz5NQ_QfCwJ_rzky7Crz4RpbCBd1DvOoLySmxsn3Aj_itg3Hd5UgAMho1xVDDbAjJ6cQjKNU3vh-E7xBJy4KhS9iP1XGzvcNy-ENLn6-giX5jNrhPZjjgY8K_IzjVrDfhKYvQlOL1wxj610VgFHNIqkb-dNbNZWvggsXPNQrw-CgrPTo3lHpxJgS2E5Cs8cJpasNssFioIjs-nMGC0MA42_N5L8wX8VkSnRbdHI9MO1JHT0hXXVUmPSxhczWoEtIgreHDyUO8cPVAiMcsLcl8nypuGKHFj)
    - [ ]  Check on [https://www.dnsbl.info/dnsbl-database-check.php](https://www.dnsbl.info/dnsbl-database-check.php)
    - [ ]  Check on [uceprotect.net](http://www.uceprotect.net/)
    - [ ]  Check on SpamHaus: [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/)
    - [ ]  Check on CloudMark: [https://csi.cloudmark.com/en/reset/?ip=165.22.223.173](https://csi.cloudmark.com/en/reset/?ip=165.22.223.173)
    - [ ]  Subscribe to JMRP: [https://kb.iweb.com/hc/en-us/articles/230267648-Subscribing-to-Microsoft-JMRP-and-SNDS](https://kb.iweb.com/hc/en-us/articles/230267648-Subscribing-to-Microsoft-JMRP-and-SNDS)
- [ ]  Use Elastic IPs as they are negotiated by Amazon and DO for whitelisting
- [ ]  Setup SPF DNS record
- [ ]  Setup DKIM DNS record
- [ ]  Setup PTR record (reverse DNS): done at [host level](http://joshua5201.github.io/blog/2015/06/06/setting-up-reverse-dns-ptr-record-in-digitalocean/): check status [here](https://mxtoolbox.com/SuperTool.aspx?action=ptr%3a159.65.157.119&run=toolpage#)
    - [ ]  [Floating IP on DigitalOcean does not support PTR](https://www.digitalocean.com/community/questions/how-do-i-set-the-ptr-for-a-floating-ip).
- [ ]  Send a test mail to see results: `echo "Testing" | mail -aReply-To:pratyushmittal@gmail.com -s "Testing Mail" check-auth@verifier.port25.com`