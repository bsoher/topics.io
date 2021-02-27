---
sort: 13
---

# Subversion Permissions

## All about SVN Permissions
In our SVN setup, all permissions are left to Apache, so configuring SVN
permissions means configuring Apache. Apache userids are totally 
separate from scion user accounts, so one doesn't need an account on 
scion to have a userid in Apache. 

Permissions are enforced by a combination of our Subversion-specific Apache
config in `/etc/apache2/conf.d/subversion.conf` and the `htdbm` file in 
`/etc/apache2/auth_groups`. The former uses the latter, and in simpler
cases the latter is the only file you'll need to change.

## Anonymous Access
As of July 2010 we opened several of our repositories to permit anonymous,
read-only access. Since this access doesn't require a password, we don't 
require it to go over HTTPS either. Both HTTP and HTTPS connections are accepted.

All anonymous access must go through the URL tree `/guest_svn`, e.g. 
`http://scion/guest_svn/vespa/`. Using different directory names for 
anonymous, read-only access and authenticated access makes the server-side
config easier for me to manage.

## Authenticated Access
The URL tree `/svn/` is reserved for authenticated access. All 
authenticated access is via `/svn/`. The reverse (all access through
`/svn/` requires authentication) is almost true, but not quite. We make
an exception for the Vespa repositories which permit anonymous read
access, even via `/svn/`.

Since authentication involves passwords, all access to the `/svn/` tree must
go over HTTPS. HTTP connections to the `/svn/` tree are rejected.

## Repository Groups
Conceptually, we have two groups of repositories. There's the Vespa 
repositories and everything else (hereafter "EE"). At present, Vespa 
repositories are: gamma, duke-mr, sample_data, vespa and vespa_docs.

Examples of EE repositories include orphans, divox, gava, idl_vespa, and libidl.

The two categories are configured differently. Vespa repositories allow 
anonymous read while write requires authentication. 

EE repositories require authentication for both read and write access. 

Furthermore, each EE repos can configured individually. For example, one
can have read access to divox, read/write access to gava, yet no access to
anything else. 

By contrast, Vespa repositories are configured as a group. If you have 
write access to one Vespa repos, you have write access to them all.


## Broad Permission Groups
SVN permissions are determined by groups that have names of the form `xxx-write`
or `xxx-read` where `xxx` is either `all`, `vespa`, or a repository name. 

The groups `all-write` and `all-read` are just what they sound like. Anyone
belonging to `all-read` has read-only access to all of our repositories, and
`all-write` adds write access to that. 

Membership in the group `vespa-write` allows read/write access to all of the
Vespa repositories. (There is no need for a `vespa-read` group since we 
already permit anonymous, read-only access to our Vespa repositories.)

Membership in the group `xxxx-write` allows read/write access to the repository
xxxx and no other. `xxxx-read` allows write access to the repository, but not
write.

## Adding or Editing a User's Membership
To add a new user or update a user's permissions, execute this command as 
root on scion --

```
#!sh 
htdbm -bt -TSDBM /etc/apache2/auth_groups <username> NULL "<group-names>:Fake password is NULL"
```

For instance, to give Iona Heap read access to everything and write access
to the sandbox, run this -- 

```
#!sh 
htdbm -bt -TSDBM /etc/apache2/auth_groups iheap      NULL "all-read,sandbox-write:Fake password is NULL"
```

Tip: to list existing permissions for all users, run this -- 

```
#!sh 
htdbm -l -TSDBM /etc/apache2/auth_groups
```

### How Permissions Are Implemented
Permissions are stored in `/etc/apache2/auth_groups` which is a DBM file.
(A DBM "file" actually consists of two files with the exensions 
`.dir` and `.pag`. Conceptually they're one file and they are conventionally 
referred to by the filename without any extension at all.)

The DBM file is a little less convenient than `/etc/apache2/.htpasswd` since
the former is not plain text and can only be manipulated through Apache's
`htdbm` utility. The command `htdbm -l` lists the contents --

```
#!sh
[root@scion conf.d]# htdbm -l /etc/apache2/auth_groups
Dumping records from database -- /etc/apache2/auth_groups
    Username                        Comment
    steinberg                       all-write:Fake password is NULL
    bsoher                          all-write:Fake password is NULL
    dtodd                           all-write:Fake password is NULL
    kyoung                          all-write:Fake password is NULL
    lana                            all-read:Fake password is NULL
    maer                            all-read,gamma-write:Fake password is NULL
    flip                            all-write:Fake password is NULL
Total #records : 7
```

You can see that each user has one entry and a comment. These files are meant
to store userids and passwords but Apache has hacked in a third feature. 
The first part of the comment (up to the first colon) is a comma-delimited
series of group names which are referred to from Apache's config file
(`subversion.conf`, in our case).

Since we're storing passwords in `/etc/apache2/.htpasswd`, the passwords in the
DBM file are meaningless and are all set to the string 'NULL'. 

If you're wondering why we are storing passwords in one place and group membership
in another, rest assured there's a good reason! 

We _could_ store passwords in the DBM file, but passwords must be passed raw 
(i.e. unhashed) to the `htdbm` utility. That means that a user would either have 
to (a) sit at a keyboard logged into scion as root in order to set his password
or (b) reveal his password to me, and I'd type it in. I don't like either scenario.
We could also edit the DBM file in the same way we edit `.htpasswd`, but the DBM
data is a mix of text and binary and it's not meant to be hand-edited. I'm not
confident that a text editor wouldn't corrupt its contents somehow.

And we _could_ store groups in a text file, but the text file syntax is keyed
by group and one lists the users in that group which is very repetitive and 
error-prone.

So storing passwords in the plain text file and groups in the DBM file gives
us the best of both worlds, even if it is a little confusing.


### An htdbm Quirk
The `-c` option claims that it creates a new database. However, if one passes
the name of an existing file, `htdbm` will merely add or update the user
although it will respond with the message `Database xxx created.` In other
words, don't rely on the `-c` option to create a fresh database for you.
