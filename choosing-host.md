# Choosing a host for setting up SMTP server

Most hosting providers block SMTP by default. They enable SMTP on servers after we drop them a mail. It can take up-to a week to get these approvals.

Considerations when choosing a host:
- Allows reverse DNS
- IPs are whitelisted

## Reverse DNS

Hosting providers that allow reverse DNS:
- AWS

Hosting providers that don't allow reverse DNS:
- DigitalOcean doesn't allow reverse DNS on elastic IP.


## Whitelisted IPs:

Most hosting providers provide an option of elastic ip which can be attached to a server. These IPs are usually whitelisted.
