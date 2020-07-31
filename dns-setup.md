# Setup up forward and reverse DNS

- Assign Elastic IP to the server. This makes it easy to change servers.
- Choose a subdomain for the mailing server: eg `mail.domain.com`
- Add `A` record for the IP pointing to `mail.domain.com`.
- File a request for setting up reverse-dns with your host: https://console.aws.amazon.com/support/contacts?#/rdns-limits
