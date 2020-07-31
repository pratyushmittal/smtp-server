# Setting up DKIM

Most of this guide is based on [DigitalOcean's guide][digitalocean guide] on DKIM setup.

DKIM is a method by which the client verifies that the content of the email were note modified. It also establishes the ownership of the domain which sends the email.

**How it works?**

The Mail Transfer Agent (MTA), i.e. Postfix, signs every outgoing email using a private key. The receiver fetches the corresponding public key from sender's DNS records and uses it to verify the content signature.

## Install DKIM

Install OpenDKIM and it’s dependencies:

```bash
sudo apt-get update
sudo apt-get install opendkim opendkim-tools
```

Edit `/etc/opendkim.conf` and add following lines to it. The explanations are given below.

```conf
AutoRestart             Yes
AutoRestartRate         10/1h
UMask                   002
Syslog                  yes
SyslogSuccess           Yes
LogWhy                  Yes

Canonicalization        relaxed/simple

ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts
InternalHosts           refile:/etc/opendkim/TrustedHosts
KeyTable                refile:/etc/opendkim/KeyTable
SigningTable            refile:/etc/opendkim/SigningTable

Mode                    sv
PidFile                 /var/run/opendkim/opendkim.pid
SignatureAlgorithm      rsa-sha256

UserID                  opendkim:opendkim

Socket                  inet:12301@localhost
```

- **AutoRestart**: auto restart the filter on failures
- **AutoRestartRate**: specifies the filter’s maximum restart rate, if restarts begin to happen faster than this rate, the filter will terminate; 10/1h - 10 restarts/hour are allowed at most
- **UMask**: gives all access permissions to the user group defined by UserID and allows other users to read and execute files, in this case it will allow the creation and modification of a Pid file.
- **Syslog, SyslogSuccess, LogWhy**: these parameters enable detailed logging via calls to syslog
- **Canonicalization**: defines the canonicalization methods used at message signing, the `simple` method allows almost no modification while the `relaxed` one tolerates minor changes such as whitespace replacement;
- `relaxed/simple` - the message header will be processed with the `relaxed` algorithm and the body with the `simple` one
- **ExternalIgnoreList**: specifies the external hosts that can send mail through the server as one of the signing domains without credentials
- **InternalHosts**: defines a list of internal hosts whose mail should not be verified but signed instead
- **KeyTable**: maps key names to signing keys
- **SigningTable**: lists the signatures to apply to a message based on the address found in the `From:` header field
- **Mode**: declares operating modes; in this case the milter acts as a signer (`s`) and a verifier (`v`)
- **PidFile**: the path to the Pid file which contains the process identification number
- **SignatureAlgorithm**: selects the signing algorithm to use when creating signatures
- **UserID**: the opendkim process runs under this user and group
- **Socket**: the milter will listen on the socket specified here, Posfix will send messages to opendkim for signing and verification through this socket; `12301@localhost` defines a TCP socket that listens on `localhost`, port `12301`


### Connect the milter to Postfix:

Edit `/etc/default/opendkim` and add the following line:

```conf
SOCKET="inet:12301@localhost"
```

Configure postfix to use this milter by editing `/etc/postfix/main.cf`. Make sure that these two lines are present in the Postfix config file and are not commented out:

```conf
milter_protocol = 2
milter_default_action = accept
smtpd_milters = inet:localhost:12301
non_smtpd_milters = inet:localhost:12301
```

Create a directory structure that will hold the trusted hosts, key tables, signing tables and crypto keys:

```bash
sudo mkdir /etc/opendkim
sudo mkdir /etc/opendkim/keys
```

Create `/etc/opendkim/TrustedHosts` and specify trusted hosts in it.
We will use this file to define both ExternalIgnoreList and InternalHosts. Messages originating from these hosts, domains and IP addresses will be trusted and signed.

Because our main configuration file declares TrustedHosts as a regular expression file (refile), we can use wildcard patters, *.domain.com means that messages coming from domain.com’s subdomains will be trusted too, not just the ones sent from the root domain.

Customize and add the following lines to the newly created file. Multiple domains can be specified, do not edit the first three lines:

```conf
127.0.0.1
localhost
192.168.0.1/24

*.domain.com

#*.domain.net
#*.domain.org
```

Create a key table at `/etc/opendkim/KeyTable`
A key table contains each selector/domain pair and the path to their private key. Any alphanumeric string can be used as a selector, in this example mail is used and it’s not necessary to change it.

```conf
mail._domainkey.domain.com domain.com:mail:/etc/opendkim/keys/domain.com/mail.private

#mail._domainkey.example.net example.net:mail:/etc/opendkim/keys/example.net/mail.private
#mail._domainkey.example.org example.org:mail:/etc/opendkim/keys/example.org/mail.private
```

Create a signing table at `/etc/opendkim/SigningTable`.
This file is used for declaring the domains/email addresses and their selectors.

```conf
*@domain.com mail._domainkey.domain.com

#*@example.net mail._domainkey.example.net
#*@example.org mail._domainkey.example.org
```

Generate the public and private keys:

```bash
cd /etc/opendkim/keys
sudo mkdir domain.com
cd domain.com

# Generate the keys
sudo opendkim-genkey -s mail -d domain.com
# -s specifies the selector and -d the domain, this command will create two files, mail.private is our private key and mail.txt contains the public key.

# Change the owner of the private key to opendkim:
sudo chown opendkim:opendkim mail.private
```

Add the public key to the domain’s DNS records:

```bash
cat mail.txt
```

It will output something like:

```bash
# illustration only - DON'T USE THIS

mail._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; "
	  "p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC5N3lnvvrYgPCRSoqn+awTpE+iGYcKBPpo8HHbcFfCI"
	  "aONa6E69qUXDAVFisYVgrKhMtInXsLB1PJFHic58pLm/OjDm4yefKXhQjM7iKzEPuBatE7O47hAx1CJpNuIdLxhILSbEmbMxJrJAG0HZVn8z6EAoOHZNaPHmK2h4UUrjOG8zA5BHfzJf7tGwI+K619fFUwIDAQAB" )  ; ----- DKIM key post for domain.com
```

It is showing the DNS record we need to add. Create a `TXT` DNS entry for `mail._domainkey` with the content as text in the brackets.

Restart Postfix and OpenDKIM:

```bash
sudo service postfix restart
sudo service opendkim restart
```

Congratulations! You have successfully configured DKIM for your mail server!

The configuration can be tested by sending an empty email to check-auth@verifier.port25.com and a reply will be received. If everything works correctly you should see DKIM check: pass under Summary of Results.

```txt
==========================================================
Summary of Results
==========================================================
SPF check:          pass
DomainKeys check:   neutral
DKIM check:         pass
Sender-ID check:    pass
SpamAssassin check: ham
Alternatively, you can send a message to a Gmail address that you control, view the received email’s headers in your Gmail inbox, dkim=pass should be present in the Authentication-Results header field.

Authentication-Results: mx.google.com;
       spf=pass (google.com: domain of contact@domain.com designates --- as permitted sender) smtp.mail=contact@domain.com;
       dkim=pass header.i=@domain.com;
```


[DigitalOcean Guide]: https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy