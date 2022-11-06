---
layout: post
title: Fix to Gmail and Nodemailer timeout problem
---

I have been using [Nodemailer](https://nodemailer.com/about/) for a few personal and commercial projects to send email from Node.js

It has been working quite fine even with Gmail. However now my Raspberry Pi memory card broke (again) and I decided to move a personal project from it to my Hetzner server. 

The email sending feature had worked nicely for a long time but after trying to setup the system to my Ubuntu server, the email sending didn't work. It always faulted to a timeout error:

```
Error: Connection timeout
    at SMTPConnection._formatError (xxx/node_modules/nodemailer/lib/smtp-connection/index.js:790:19)
    at SMTPConnection._onError (xxx/node_modules/nodemailer/lib/smtp-connection/index.js:776:20)
    at Timeout.<anonymous> (xxx/node_modules/nodemailer/lib/smtp-connection/index.js:235:22)
    at listOnTimeout (node:internal/timers:564:17)
    at process.processTimers (node:internal/timers:507:7) {
  code: 'ETIMEDOUT',
  command: 'CONN'
}
```

The old code used directly the available `service: 'gmail'` transport setting:

**Non-working solution:**
```js
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: settings.notification_email_address,
    pass: settings.notification_email_password
  }
})

//Create mail settings
const options = {...}

//Send 
const res = await transporter.sendMail(options);
``` 

I tried to find a solution to this problem for a while. Finally after testing many things, such as opening ports or disabling firewall, [this quite new StackOverflow answer](https://stackoverflow.com/a/73996385/8140625) helped!

For some reason, everything works fine when using SMTP settings instead of Gmail service. The following example does the trick on my Ubuntu server:

**Working solution:**
```js
const transporter = nodemailer.createTransport({
  host: "smtp.gmail.com",
  port: 587,
  secure: false,
  requireTLS: true,
  auth: {
    user: settings.notification_email_address,
    pass: settings.notification_email_password
  }
})

//Create mail settings
const options = {...}

//Send 
const res = await transporter.sendMail(options);
```

Hopefully this helps somebody!