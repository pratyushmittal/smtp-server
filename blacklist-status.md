# Check IP status on common spam tracking lists

- [ ]  Check on [http://www.anti-abuse.org/multi-rbl-check/](http://www.anti-abuse.org/multi-rbl-check/)
- [ ]  Check on [https://www.dnsbl.info/dnsbl-database-check.php](https://www.dnsbl.info/dnsbl-database-check.php)
- [ ]  Check on [uceprotect.net](http://www.uceprotect.net/)
- [ ]  Check on SpamHaus: [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/)
- [ ]  Try sending a mail to some @icloud.com user. Request for whitelisting on CloudMark if blocked: [https://csi.cloudmark.com/en/reset/](https://csi.cloudmark.com/en/reset/)
- [ ]  Send an email to outlook user. Request for [whitelisting](https://support.microsoft.com/en-us/supportrequestform/8ad563e3-288e-2a61-8122-3ba03d6b8d75) if blocked.


## Template for blacklist removal request

Sometimes we need to provide details about how we protect spam.

```txt
[My company name] does [this and this].
It sends emails for [this and this purpose].

Protection:
- All emails follow with SPF and DKIM specifications.
- All emails contain "List-Unsubscribe" headers for one-click unsubscribe.
- Have enabled SASL and firewall to send emails only from internal IPs.
- Have registered on Google Postmaster and Outlook's SNDS to track IP reputation.
- All complaints and bounces are immediately dealt with.

Quality:
- We email only registered users who ask for email updates.
- We have not purchased any email list.
```

Sometimes we need to provide details about what has changed:

```txt
Hi,

It seems <service provide> is blocking emails from our email server with IP <this.this.this.this>.

We shifted hour hosting to <this-this-host> on <this-date>. We have enabled firewalls and taken all the protection to prevent any spam from this ip.
```