---
title: "Sending Emails using Django and Sendgrid"
date: 2017-08-10
categories: 
  - "tech"
---

Recently, i setup a django app which uses  [sendgrid](http://sendgrid.com) to send emails. I will go through the steps in this short blog.

1. Register at [sendgrid](http://sendgrid.com)
2. Choose SMTP for sending emails
3. Get an API key. The last step will redirect you to this. This key will also be your password. The username that they gave me was _apikey_ so, i guess this remains same for everyone.
4. Configure your django settings to this: _EMAIL\_HOST\_USER = '<your username here>'_ _EMAIL\_HOST = 'smtp.sendgrid.net'_ _EMAIL\_HOST\_PASSWORD = '<your password here>'_ _EMAIL\_PORT = 587_ _EMAIL\_USE\_TLS = True_ _EMAIL\_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'_  (this is the default value of EMAIL\_BACKEND btw)
5. Use _django.core.mail.send\_mail_ for sending emails now
