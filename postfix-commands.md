# Useful Postfix commands for debugging and monitoring

## Sending and receiving mails

- Sending test mail: `echo "Testing" | mail -s "Testing Mail" yourname@gmail.com`
- Reading incoming messages: `mail` navigation: [https://unix.stackexchange.com/questions/26790/what-is-mail-and-how-is-it-navigated](https://unix.stackexchange.com/questions/26790/what-is-mail-and-how-is-it-navigated)

### Handling pending queue

- see messages in queue: `mailq` or `postqueue -p`
    - Delete messages in mailq: `postsuper -d ALL`


## Checking logs or debugging any email

- inspecting particular message in queue: `postcat -vq XXX_MESSAGE_ID`
- Seeing error logs:
    - `tail -f /var/log/mail.err`
    - `/var/log/mail.log | grep error`
- for generating summary of logs: `pflogsumm /var/log/mail.log -d today`
- Checking logs for particular email: `cat /var/log/mail.log | grep user@email.com`


## Configuration related

- Check postfix conf: `postconf`
- For default settings: `postconf -d`
- Check postfix differences from default: [https://gist.github.com/robinsmidsrod/1339247](https://gist.github.com/robinsmidsrod/1339247)

```bash
cd ~/
postconf -d >postfix.defaults
postconf >postfix.conf
diff -u postfix.defaults postfix.conf | grep -v '^ '|grep -v '^@' |grep -v '^---' | grep -v '^+++'
```

- Change host name via (updates system hostname): `sudo hostnamectl set-hostname mail.mydomain.com`
