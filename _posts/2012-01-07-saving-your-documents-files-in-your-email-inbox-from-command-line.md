---
title: Saving your documents, files in your email inbox from command line
date: 2012-01-07 16:07:05.000000000 +05:30
---

Whenever I reinstall my system, I wish i could just take backup of some small important files, So I wrote this python script which composes a email with compressed attachments and sends it to my email (support for smtp is required). If you use any popular or common email provider a nice side effect is that you have two copies of the same data as most email provider archives, emails they relay.

Download the python script for all platforms here (<a href="https://github.com/linuxexp/backup2mail">https://github.com/linuxexp/backup2mail</a>)

## Dependencies

The script is a python one, so python interrupter is required other than that script depends on smtplib, email, tar modules.

Any mainstream linux distro will have all dependencies meet, so just download and chmod

## Parameters
`./mail.py --help`

This'll show bunch of options, but don't get scared way you won't need it. you'll only need to use --dir or -d to specify the directory to compress and send. You can use other options if you are using muti-user system.

## Configure
You need to configure the script for your email provider on your first use. edit the python file as follows, this will be used as default if parameters are specified.

```python
smtp_server = "smtp.gmail.com"
smtp_port = "587" # port may be 465 or 587, check with mail provider
smtp_login = ["example@gmail.com","password"] # example@example.com, password
mail_meta =  ["from@gmail.com", "to@gmail.com", "Backup"] # from, to, subject
mail_text = "Backup content" # some text 
```

Things are pretty self explanatory, you can check with your email provider for the corresponding values.

## GMAIL filters
If you're using GMAIL you can create a email filter which will automatically move you email under certain criteria to specific label.

**Note**

The file(s) are sent as compressed attachments, so attachment size limits of the email provider will apply, for GMAIL it's 25 MB as of this post.

## Compression

The program compresses the folder supplied to bz2 archive using Burrows–Wheeler algorithm. if you are compressing textual stuff( sources, web projects, xml ...) then there'll be considerable compression. for i.e in compressing 30 MB of<strong> textual stuff</strong> you can expect it to result into something around 8 MBs.