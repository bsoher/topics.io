---
sort: 3
---

# Subversion Administrations

## So You Want To Be A Subversion Administrator
This is a guide to some common Subversion tasks. It doesn't cover users
and permissions which is a subject that 
[UsersPasswordsAndPermissions has its own documentation].

This guide is no substitute for understanding Subversion and Apache
config. That said, this attempts you to tell you what you need to know to make
simple changes.

## Creating A New Repository
I wrote a script for creating new repositories. To run it, you'll
need to log on to scion as root. The script is in `/data/svn/` and
it is called `create_new_svn_repos.py`. Run it like so:

```
 python create_new_svn_repos.py name_of_new_repository
```
 
If you want to see exactly what it is doing, have a look at the 
script. It's short.

Note that it does not create the directories `trunk`, `branches` 
and `tags`. You'll need to do that yourself.

You will also need to add an entry into `/data/vespa/scion_admin/apache_conf/subversion.conf` (which 
is referenced from `/etc/apache2/conf.d).

1. Duplicate one of the entries therein and change the repository name as
 appropriate.
1. Restart apache (`>apachectl graceful`) so it notices the new config --
1. You might want to [SubversionMailerConf set up commit notifications for the new repository].

Note. If you want the new repository to be available read-only via the guest_svn symlink (which is how 
the Vespa repositories are accessed read-only) you need to carefully add the repository name to two lists
of names in the subversion.conf file.  There is info about these lists in the doc strings in that file.

If you make changes to the subversion.conf file in your local working copy of the scion_admin repository
you need to commit from your local copy, then log into scion, update the repository at /data/vespa using
'>svn up scion_admin' command. 

While you will not need to add the repository to the individually set ones at the bottom of this
file, you *will* need to restart apache server for changes to take effect.
 
 
## Deleting a Repository
Just delete the directory --

```
#!sh
rm -rf /data/svn/name_of_the_repository
```
  

## Renaming a Repository
Subversion repositories don't know their own names, so renaming
the repository directory renames the repository.

If the repos is referenced anywhere in our config files
(`scion_admin/apache_conf/subversion.conf` is a likely candidate), you'll
need to change those references. A utility like `grep` is a good way to
find all the references you might need to change.


