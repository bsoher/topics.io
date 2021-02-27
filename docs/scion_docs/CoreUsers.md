---
sort: 6
---

# Core Users
Core users are the people who have deity-level access to every aspect of the 
project. Currently, they are Brian, Karl, David and me.

Core users follow all the rules of regular users, so they need to follow the
normal procedure for adding a user to SVN & Trac. This document is only 
concerned with extra or special procedures related to core users.

## Adding or Removing
Core users and their email addresses are hardcoded in two places.

### `/data/vespa/scion_admin/mailer.conf`
This file controls how SVN sends out commit notifications. You can ignore
most of the file; the important part (email addresses) are near the bottom,
after the phrase, "Repository-specific config goes here."

Since no one is required to get commit notifications for any repository,
there's no guarantee any user (core or otherwise) will be listed in this
file. But core users probably will be so you need to check. 

The format of the file is straightforward and explained therein.

Note that an address might be present multiple times.


### `/data/vespa/scion_admin/create_new_trac_instance.py`
Core users and their email addresses are hardcoded in a dict at the top
of the file.

## Changing an Email Address
When a core user's preferred email address changes, you need to notify both
SVN and Trac.

As described above, SVN's email config is all in `mailer.conf`. 

Trac is trickier -- email addresses are not centralized. They're in the 
database associated with each instance. I wrote a script to handle email 
address changes; it is `/data/vespa/scion_admin/change_email_address.py`. Invoke it
like so to change the email address for user `fred` to `fred@example.com`:
```
/data/vespa/scion_admin/change_email_address.py fred "fred@example.com"
```

## When You Are Done Editing Files in scion_admin
The files in the 'scion_admin' local repository have to be committed. Then you have to log into Scion and go to the /data/vespa/scion_admin directory and update the files there from the repository.  You do this from the command line by:

changing over to root (>sudo -s) 

running >svn up
