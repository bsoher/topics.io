# The Quick Guide to Adding a New User
Note that most of our Trac instances and SVN repositories are readable by
everyone. In other words, if someone only needs read access to something
that already permits anonymous read access, you don't need to add them as 
a user.

This makes reference concepts in our SVN and Trac administration guides. If 
you haven't read them, some of this will be mysterious but it should work
anyway.

The examples below are for Car Talk's Director of Grad Student Transportation,
Iona Heap (userid `iheap`).

1. Get Iona to run the following command and send you the single line of 
 output.
```
 htpasswd -nm iheap
```
 `htpasswd` is available anywhere Apache is installed. This is by default on OS X;
 on other systems you'll have less luck. If it doesn't work on the user's 
 computer, email them the HTML file attached to this page and have them open it
 locally.
1. Log on to scion as root. You're going to edit `/etc/apache2/.htpasswd`, but
 first you'll make a backup. 
1. Copy `/etc/apache2/.htpasswd` to `/etc/apache2/.htpasswd.yymmdd` where `yymmdd`
 is the current year, month and date. For example, on 16th of February 2010 --
```
 cp   /etc/apache2/.htpasswd   /etc/apache2/.htpasswd.100216
```
1. Use a text editor (e.g. `vim`) to add the `htpasswd` output from step 1
 to `/etc/apache2/.htpasswd`. 

 Once you'd done that, Iona gains the ability to authenticate
 herself to SVN and Trac. To SVN, this is meaningless until she also belongs
 to some authentication groups (see below). To Trac, authentication gives
 one write access to most of our Trac instances. (This is configured on an
 instance-by-instance basis.)
1. To enable some SVN access, you alter `/etc/apache2/auth_groups`. You must
 do this with the `htdbm` application; it's not possible to do it with a 
 text editor. 
 
 First, you need to decide to what groups
 the person will belong. The `all-read` group gives her read-only access to 
 all repositories. The `vespa-write` group gives her read/write access to
 the various Vespa repositories. For more details, see SubversionPermissions.
 In the example below, we'll add Iona to the `all-read` group.
```
 htdbm -bt -TSDBM /etc/apache2/auth_groups iheap NULL "all-read:Fake password is NULL"
```
 This command allows Iona to write to libidl and all of the vespa repositories -- 
```
 htdbm -bt -TSDBM /etc/apache2/auth_groups iheap NULL "libidl-write,vespa-write:Fake password is NULL"
```
