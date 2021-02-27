---
sort: 12
---

# Users Passwords and Permissions

## SVN and Trac: Users, Passwords and Permissions
There are two levels to consider: authentication and permissions. 

## Authentication
Authentication means answering the question "Is your userid in my database and
do you have the correct password?" Since all access to SVN and Trac is through
HTTPS, authentication is handled by Apache. 

Users and their passwords are stored in `/etc/apache2/.htpasswd`. This is an
ordinary text file in which each user and his password are stored on a single
line. If you're not in this file, you don't exist. For security reasons, this file is 
not backed up.

Adding a new userid/password is a three step process.

1. Get the user to run the `htpasswd` utility which is installed with
    Apache. It's available by default under OS X and a few Linuxes. The
    specific command is `htpasswd -nm username`. (See example below.)
    If 
    `htpasswd` isn't available to a user, one can download the
    attached Web page and run it locally.) 
1. Have the user mail the output of the command (which is just one line) to 
    you.
1. Add that line to `/etc/apache2/.htpasswd` (after making a backup of that
    file, of course).

For instance, suppose we wanted to add a line for Car Talk's Director of Grad
Student Transportation, Iona Heap. Iona (userid `iheap`) would run this: 
```
$ htpasswd -nm iheap 
New password: 
Re-type new password:
iheap:$apr1$k/RiPbvx$J8gVhdZq.Q5wmGxWzmYPu1
```

That last line is the output of `htpasswd` and is the line to add to
`/etc/apache2/.htpasswd`. It's a one-way hash of Iona's password (which is
'cartalk'). That means Iona's password isn't revealed to me and can't be
sniffed in transit (assuming it's a decent password).

Note that the username is part of the hash that's created (the stuff to
the right of the colon) so if you want to change the username, you have
to re-run `htpasswd`. It's not sufficient to just change the string to
the left of the colon.


## Permissions
Permissions matter only once a user has been authenticated. Permissions answer
the questions, "To what resources are you permitted access and what are you 
allowed to do with them?"

SVN and Trac have separate permissions. The only place where they collide is
in the unusual scenario where you want to give someone access to a Trac instance
without giving them read access to the corresponding Subversion repository. Trac
browses as the apache user and so it can see everything in a repository.
Therefore, a user who has BROWSER_VIEW permission in a Trac instance will be
able to browse the associated SVN repository even if you have not granted him
read permission to that repository. Currently, I don't have a solution 
for this problem because it hasn't come up in real life. 


### SVN Permissions
SVN permissions have their own document: SubversionPermissions

### Trac Permissions
Trac permissions have their own document: OurTracPermissions
