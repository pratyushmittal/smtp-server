# Preparing SMTP server for production

Now we have our server ready. Try sending a simple test mail to your `gmail.com` address.

```bash
echo "Testing" | mail -s "Testing Mail" your@gmail.com
```

It will probably land in the spam folder because we are not using DKIM verification for above. Don't worry. **Important thing is that it should deliver.**

