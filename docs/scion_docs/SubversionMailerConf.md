---
sort: 2
---

# Subversion Mailer Config

## SVN Commit Notifications
This document tells you how to add, edit and delete entries in the file that
controls the emails that Subversion sends when someone commits a change to SVN.

## The Background
Subversion offers lots of hooks into its process, and one of those is the 
"post-commit" hook. We have implemented this hook as a shell script that 
does a couple of things. One of those things is to call `mailer.py` which is
a standard script distributed with Subversion. 

That script is driven by the data in the file `/data/svn/mailer.conf` (the
same directory as `mailer.py`). 
*If you need to make changes, you need to edit `mailer.conf`.* We have
never needed to edit `mailer.py`, and you probably won't need to either.

## mailer.conf
The section that you want to change contains email addresses and repository 
information. It's at the
end of this file after a collection of global settings (like SMTP gateway name,
subject line length limit, etc.)

The file itself contains good instructions on how this section of the config
file works, but I'll repeat them here for clarity.

Email rules are expressed in sections or groups (I'll use the latter term)
that follow a INI-file style syntax. Strangely enough, the name of the group 
doesn't matter (SVN ignores the name) so I just use whatever names are 
meaningful to me.

Each group has two (or three) key/value pairs in it. Here's an example with
two:

```
[gamma]
for_repos = /data/svn/gamma$
to_addr = philip@example.com delicasso@example.net maer@example.ch
```

The `for_repos` line is a regular expression that matches against the 
fully-qualified path of the repos, like `/data/svn/gamma`, `/data/svn/vespa`,
`/data/svn/vespa_docs`, or `/data/svn/idl_vespa`. In the example above, the
dollar sign at the end of `/data/svn/gamma$` says that the matching string must
end there. In other words, that regex matches `/data/svn/gamma` but not 
`/data/svn/gamma_docs`.

The `to_addr` line is hopefully self-explanatory.

The third line that might appear in a group is the `commit_url`. This 
refers to the URL that appears in the email that SVN sends. There's a global 
setting for this (in `mailer.conf`) which works fine for any repository in 
which the repos name (e.g. `gamma`) is the same as the URL of the Trac instance 
with which the repos is associated. That's true for all of our repositories 
except the main one (`vespa`), so that repos group gets a custom `commit_url`:

```
[vespa]
for_repos = /data/svn/vespa$
to_addr = philip@example.com bsoher@example.com karl.young@example.edu
commit_url = http://scion.duhs.duke.edu/vespa/project/changeset/%(rev)s/ 
```

Last but not least, here's an example of a group with a more complex regex.
The regex below matches `/data/svn/gamma_docs` and `/data/svn/vespa_docs`.

```
[gamma_and_vespa_docs]
for_repos = /data/svn/(?:gamma|vespa)_docs$
to_addr = philip@example.com bsoher@example.com delicasso@example.net
```

As a final note, I'll point out that our config is quite simple and it might 
lead you to think that SVN automatically only matches one group per commit. 
This is how we've configured things, but it doesn't happen this way 
automatically.

For instance, consider the following configuration, and note that the 
`for_repos` regexes lack a trailing `$`. 

```
[vespa]
for_repos = /data/svn/vespa
to_addr = foo@example.com

[vespa_docs]
for_repos = /data/svn/vespa_docs
to_addr = bar@example.com
```

When someone commits a change to `vespa`, `foo@example.com` will get a change
notification as intended. But when someone commits to `vespa_docs`, both the
regex `/data/svn/vespa` and the regex `/data/svn/vespa_docs` match. (Try it
out in your favorite regex explorer if you don't believe me.) As a result,
both `foo@example.com` and `bar@example.com` will get change notifications,
and the former's email will contain the wrong commit URL. 

The moral of the story is that regexes are slippery beasts and you'd be wise
to test them well before using them in this file.

## When You Are Done Editing Files
The files in the 'scion_admin' repository have to be committed. Then you have to log into Scion and go to the /data/vespa/scion_admin directory and update the files there from the repository.  You do this from the command line by:

changing over to root (>sudo -s) 

running >svn up
