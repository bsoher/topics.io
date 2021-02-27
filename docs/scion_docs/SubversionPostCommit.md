---
sort: 4
---

# Subversion Post-Commit

## The Subversion Post-Commit Hook
Subversion offers the ability to execute a custom, server-side script after a commit has completed. We take advantage of that in all of our repositories. This document describes how the script is wired up on scion.

The `scion_admin` script `create_new_svn_repos.py` automatically enables and connects the our post-commit script for you. If you use that script to create a repository, then the post-commit hook should work automatically. Because of that, this document doesn't contain any steps you need to carry out or work you need to do. It's just informative. 

## Background - What's a Hook?
Subversion offers the ability to "hook" certain events to allow the execution of a custom server-side script. [The Subversion documentation of hooks](http://svnbook.red-bean.com/en/1.6/svn.reposadmin.create.html#svn.reposadmin.create.hooks) describes it this way:

   A hook is a program triggered by some repository event, such as the creation of a new revision or the modification of an unversioned property. Some hooks (the so-called 'pre hooks') run in advance of a repository operation and provide a means by which to both report what is about to happen and prevent it from happening at all. Other hooks (the 'post hooks') run after the completion of a repository event and are useful for performing tasks that examine - but don't modify - the repository. Each hook is handed enough information to tell what that event is (or was), the specific repository changes proposed (or completed), and the username of the person who triggered the event.

## Hook Activation
One activates a hook by adding a script to a repository's `hooks` directory. All of our repositories are configured in the same way -- each implements a post-commit hook, and it's the same script for each repository.

For instance, we can see in the `vespa` repository that `post-commit` points to `post-commit.sh` in the `scion_admin` directory. (The `.tmpl` files are templates created by Subversion when the repository was created. They're inert models only; they're not executed.)

```
$ ls -l /data/svn/vespa/hooks/
total 36
lrwxrwxrwx 1 www-data www-data   38 Apr 24 09:59 post-commit -> /data/vespa/scion_admin/post-commit.sh
-rw-r--r-- 1 www-data www-data 2022 Apr 22 11:56 post-commit.tmpl
-rw-r--r-- 1 www-data www-data 1663 Apr 22 11:56 post-lock.tmpl
-rw-r--r-- 1 www-data www-data 2344 Apr 22 11:56 post-revprop-change.tmpl
-rw-r--r-- 1 www-data www-data 1592 Apr 22 11:56 post-unlock.tmpl
-rw-r--r-- 1 www-data www-data 3510 Apr 22 11:56 pre-commit.tmpl
-rw-r--r-- 1 www-data www-data 2410 Apr 22 11:56 pre-lock.tmpl
-rw-r--r-- 1 www-data www-data 2818 Apr 22 11:56 pre-revprop-change.tmpl
-rw-r--r-- 1 www-data www-data 2100 Apr 22 11:56 pre-unlock.tmpl
-rw-r--r-- 1 www-data www-data 2852 Apr 22 11:56 start-commit.tmpl
```

The `hooks` directory of each repository looks exactly like the one above. 

NB. In migrating to scion2 (July 2018) I found that I needed to make post-commit.sh in the /data/vespa/scion_admin directory executable. 

```
sudo chmod 755 /data/vespa/scion_admin/post-commit.sh
```

Otherwise I got a "Warning: post-commit hook failed (exit code 255) with no output." every time I checked in a repository.

## Post-Commit Hook Implementation
As of this writing, our post-commit shell script hook runs two Python scripts. The first is a standard script that many (most?) Subversion installations use. It's `mailer.py`, the Subversion-supplied script that sends commit notifications.

The second script is entirely custom. It's `svn_cop.py` which checks for the presence of 
`from __future__ import division` and also the presence of tabs (ASCII 9).