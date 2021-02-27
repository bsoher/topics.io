---
sort: 7
---

# Sending Mail from Scion

## How (and Why) Email is Sent from Scion
We send Trac and SVN notifications from scion. They're sent programmatically 
from Python. To send mail, one must use an SMTP server. Scion does *not* 
run its own SMTP server. Instead, it uses one provided by Duke at 
`smtpgw.duhs.duke.edu`.

This SMTP server allows anonymous connections. Accepting anonymous 
connections can be a little dangerous, so the admins of this server take
precautions to ensure that they know who is using it to send mail. Before
using this server, one must register the sender.

## Registering Scion To Send Email
Scion is already registered to send email via `smtpgw.duhs.duke.edu`, but
those registrations age out (every 6-12 months?) and must be renewed. 

Registration happens here: https://email.duhs.duke.edu/

Click on "DUHS SMTP Gateways". Note that that link will fail if you're
accessing the site from outside the Duke network.

You'll be asked to fill out a form that's not too difficult. At the end
they ask for contact info. A confirmation email will be sent to the email
address you provide, and that email address will supposedly be sent a 
warning a week or two before the registration ages out. 

## Testing Mail
If you want to test sending mail from scion, here's a short Python script
that will do it for you. You don't need to execute it as root.

```

import smtplib

recipient = "philip@semanchuk.com"
sender = "no_one@example.com"
message = "hello world"
server = smtplib.SMTP("smtpgw.duhs.duke.edu")
server.sendmail(sender, recipient, message)
server.quit()

```
