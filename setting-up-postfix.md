# Installing postfix and setting up authentication

Much of this guide is based on [Ubuntu's installation guide for Postfix][ubuntu guide].

## Terms to understand during installation

```bash
myorigin = mydomain.com 
# The domain name to append when the UNIX user sends out a mail. (For eg. If UNIX user john sends mail, then john@mydomain.com will be the sender address)

myhostname = mail.mydomain.com
# The unique FQDN (fully-qualified-domain-name) of your mail server. When talking to other SMTP servers, it identifies itself as mail.mydomain.com

mydestination = mydomain.com mail.mydomain.com
# You are instructing postfix to receive mails for the domains mydomain.com mail.mydomain.com, whose valid recipients can be specified using local_recipient_maps

# FQDN means domain name including the hostname
# so FQDN is mail.mydomain.com
```

## Install and configure postfix

### Install
Simply accept the defaults when the installation process asks questions.
```bash
sudo apt-get install postfix
```

### Configure

Run configuration using:
```bash
sudo dpkg-reconfigure postfix
```

Use these details when asked:
1. General type of mail configuration: **Internet Site**
2. NONE doesn't appear to be requested in current config
3. System mail name: **mydomain.com**
4. Root and postmaster mail recipient: **<admin_user_name>**
5. Other destinations for mail: **mail.mydomain.com, mydomain.com, localhost.mydomain.com, localhost**
6. Force synchronous updates on mail queue?: **No**
7. Local networks: **127.0.0.0/8**
8. Yes doesn't appear to be requested in current config
9. Mailbox size limit (bytes): **0**
10. Local address extension character: **+**
11. Internet protocols to use: **all**

### Configuring postconf for authentication

The postfix configuration is stored in the file `/etc/postfix/main.cf`. We can edit it manually or change values in it using `sudo postconf -e <value_to_change>`.

We will probably be connecting to this SMTP server from another machine. We need to setup authentication to allow connections with proper passwords.

Postfix doesn't implement its own authentication. It uses the "simple authentication and security layer" provided by the system. This is called SASL.

We configure postfix to use SASL (saslauthd) for authentication.

```bash
sudo postconf -e 'smtpd_sasl_local_domain ='
sudo postconf -e 'smtpd_sasl_auth_enable = yes'
sudo postconf -e 'smtpd_sasl_security_options = noanonymous'
sudo postconf -e 'broken_sasl_auth_clients = yes'
sudo postconf -e 'smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination'
sudo postconf -e 'inet_interfaces = all'
```

Next edit `/etc/postfix/sasl/smtpd.conf` and add the following lines:

```conf
pwcheck_method: saslauthd
mech_list: plain login
```

Generate certificates to be used for TLS encryption and/or certificate Authentication:

```bash
touch smtpd.key
chmod 600 smtpd.key
openssl genrsa 1024 > smtpd.key
openssl req -new -key smtpd.key -x509 -days 3650 -out smtpd.crt # has prompts
openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 3650 # has prompts
sudo mv smtpd.key /etc/ssl/private/
sudo mv smtpd.crt /etc/ssl/certs/
sudo mv cakey.pem /etc/ssl/private/
sudo mv cacert.pem /etc/ssl/certs/
```

Configure Postfix to do TLS encryption for both incoming and outgoing mail:

```bash
sudo postconf -e 'smtp_tls_security_level = may'
sudo postconf -e 'smtpd_tls_security_level = may'
sudo postconf -e 'smtpd_tls_auth_only = no'
sudo postconf -e 'smtp_tls_note_starttls_offer = yes'
sudo postconf -e 'smtpd_tls_key_file = /etc/ssl/private/smtpd.key'
sudo postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/smtpd.crt'
sudo postconf -e 'smtpd_tls_CAfile = /etc/ssl/certs/cacert.pem'
sudo postconf -e 'smtpd_tls_loglevel = 1'
sudo postconf -e 'smtpd_tls_received_header = yes'
sudo postconf -e 'smtpd_tls_session_cache_timeout = 3600s'
sudo postconf -e 'tls_random_source = dev:/dev/urandom'
sudo postconf -e 'myhostname = server1.example.com' # remember to change this to yours
```

The file `/etc/postfix/main.cf` should now look like this:

```conf
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

myhostname = server1.example.com
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = server1.example.com, example.com, localhost.example.com, localhost
relayhost =
mynetworks = 127.0.0.0/8
mailbox_command = procmail -a "$EXTENSION"
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
smtpd_sasl_local_domain =
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
smtpd_tls_auth_only = no
#Use these on Postfix 2.2.x only
#smtp_use_tls = yes
#smtpd_use_tls = yes
#For Postfix 2.3 or above use:
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_key_file = /etc/ssl/private/smtpd.key
smtpd_tls_cert_file = /etc/ssl/certs/smtpd.crt
smtpd_tls_CAfile = /etc/ssl/certs/cacert.pem
smtpd_tls_loglevel = 1
smtpd_tls_received_header = yes
smtpd_tls_session_cache_timeout = 3600s
tls_random_source = dev:/dev/urandom
```

Restart the postfix daemon like this:

```bash
sudo /etc/init.d/postfix restart
```

### Setup SASL

We have configured postfix to use SASL. But now we have to configure SASL itself.

Install SASL related libraries:
```bash
sudo apt-get install libsasl2-2 sasl2-bin libsasl2-modules
```

We not need to move root path of `saslauthd` into `/var/spool/postfix`.

First, we edit `/etc/default/saslauthd` in order to activate saslauthd. Remove # in front of `START=yes` Or replace with `START=no`, add the `PWDIR`, `PARAMS`, and `PIDFILE` lines and edit the `OPTIONS` line at the end:

```conf
# This needs to be uncommented before saslauthd will be run automatically
START=yes

PWDIR="/var/spool/postfix/var/run/saslauthd"
PARAMS="-m ${PWDIR}"
PIDFILE="${PWDIR}/saslauthd.pid"

# You must specify the authentication mechanisms you wish to use.
# This defaults to "pam" for PAM support, but may also include
# "shadow" or "sasldb", like this:
# MECHANISMS="pam shadow"

MECHANISMS="pam"

# Other options (default: -c)
# See the saslauthd man page for information about these options.
#
# Example for postfix users: "-c -m /var/spool/postfix/var/run/saslauthd"
# Note: See /usr/share/doc/sasl2-bin/README.Debian
#OPTIONS="-c"

#make sure you set the options here otherwise it ignores params above and will not work
OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"
```

Next, we update the dpkg "state" of `/var/spool/postfix/var/run/saslauthd`. The saslauthd init script uses this setting to create the missing directory with the appropriate permissions and ownership:

```bash
sudo dpkg-statoverride --force-all --update --add root sasl 755 /var/spool/postfix/var/run/saslauthd
```

This may report an error that "--update given" and the "/var/spool/postfix/var/run/saslauthd" directory does not exist. You can ignore this because when you start saslauthd next it will be created.

Finally, start saslauthd:

```bash
sudo /etc/init.d/saslauthd start
```

### Testing

To see if SMTP-AUTH and TLS work properly now run the following command:

```bash
telnet localhost 25
```

After you have established the connection to your postfix mail server type `ehlo localhost`.

If you see the lines

```
250-STARTTLS
250-AUTH
```
among others, everything is working.

Type `quit` to return to the system's shell.


## Creating SASL username and password

Now we only need to create username and password to connect another machine with this smtp server for sending emails.

Since we are using sasl, we only need to add users in linux for authentication.

```bash
sudo adduser rockstar
SOME333RANDOM222password

# to change password
# sudo passwd rockstar

sudo /etc/init.d/saslauthd restart
sudo /etc/init.d/postfix restart
```

### Connecting from another machine and sending mail

We will connect and authenticate using telnet on another machine.

Run following on the machine from which we will be connecting to our SMTP server.
```bash
# generate base64 hash of username and password
perl -MMIME::Base64 -e 'print encode_base64("rockstar");'
perl -MMIME::Base64 -e 'print encode_base64("SOME333RANDOM222password");'

telnet server.com 25
ehlo mydomain.com
auth login
{{ hash of username }}
{{ hash of password }}

# if auth fails, then you might need to escape special chars in password

mail from: username@example.com
rcpt to: friend@hotmail.com, friend2@yahoo.com
data
Subject: Test mail

Hello,

This is an email sent by using the telnet command.
.

# quit telnet
quit

# If sending succeeds but mail does not arrive, check logs or pflogsumm
# try in main.cf: always_add_missing_headers = yes 
```

If you need any help on sending email using telnet, use [MediaTemple's Guide](https://mediatemple.net/community/products/dv/204404584/sending-or-viewing-emails-using-telnet).


## Troubleshooting

- Check the original [Ubuntu's guide for postfix](https://help.ubuntu.com/community/Postfix#Troubleshooting).
- Check [common Postfix commands](./postfix-commands.md) for debugging logs.


[ubuntu guide]: https://help.ubuntu.com/community/Postfix
