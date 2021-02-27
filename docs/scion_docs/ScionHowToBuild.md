---
sort: 10
---

# How to Rebuild Scion

## Building a New Server
This is an outline for how to build a new server for hosting Vespa. It's a good
starting document for step-by-step instructions
for building a new server, upgrading the existing one, or moving it to new 
hardware or a new operating system.

As of this
writing (May 2012), our server is `scion.duhs.duke.edu` and it runs Ubuntu 
Desktop 12.04 LTS. It was upgraded from RHEL 5 just last month. For
documentation's sake, I'll note that Ubuntu 12.04 offers Python 2.7.3 and 
SVN 1.6.17. We installed Trac 0.12.3 by hand (see below).


Scion has two main functions. First, it hosts our SVN (Subversion) 
repositories. Second, it hosts our Web site. The Web site has a grand total 
of one (1) static page (the page at the root URL). All other pages are served
through Trac.

```
#!div style="padding: 1em; border: 1px solid #2B73D7; background-color: #ffd; margin-bottom: 1em;"

Note: In the event of a catastrophic server failure (e.g. disk crash), you will probably not have access to this document when you need it. I recommend that every time you update it, print it (to PDF or paper) and stash that copy somewhere.
```


# Background Information
## File Layout
Our files live entirely in the `/data` directory. There you'll find 
directories for SVN, Trac and a third called vespa. In this latter directory
reside awstats (see below) and a very important directory called `scion_admin`. 
All of our custom scripts and config files reside in `scion_admin`.

The `scion_admin` directory is SVN-managed. That is, the files 
there got there via an `svn co` command, and when you edit them you are 
expected to commit them with an appropriate change comment. (The files are
stored in
[//svn/scion_admin an SVN repos called scion_admin], appropriately enough.)

Instead of copying the files from `scion_admin` to their appropriate locations on the server, we instead use Unix symlinks to refer to the `scion_admin` files from wherever they need to be. For instance, the file `mailer.conf` needs to live in `/data/svn`. Instead of copying it there, we create a symlink in `/data/svn` that points to the real file in `scion_admin`.

Our scripts and config files contain lots of dependencies on the file layout described below. If you decide to follow a different layout (for instance, by using `/var/data` instead of `/data`), you'll have to review the files in `scion_admin` to see if they're affected by that change.

## Assumptions
This document assumes the following:

 * The host operating system already has Apache installed
 * You have access to our Trac and SVN backups
 * You have backups of `/etc/apache2/.htpasswd` and `/etc/apache2/auth_groups.*`. (It's no big deal if you don't have these files, but you'll need to set up a new password for everyone who needs access to SVN and/or Trac.)
 * You are logged in as root

## About Backups
We [BackupOverview  back up each Trac instance and SVN repos daily], and we 
keep as many days of backups as we have space for. The backup time is 
embedded in the filename. For instance, the
files below are a backup of the SVN repos _sandbox_ created on 2 May 2012 
at 5:11 AM, and a backup of the Trac instance _gamma_ created on 17 April
2012 at 8:10AM. 

```
svn.sandbox.2012-05-02.05.11.04.dump.bz2
trac.gamma.2012-04-17.08.10.35.tar.bz2
```


At present, there's over 1,000 files in the backup directory. 

I wrote a script called `grab_latest_backups.py`. It finds and copies the 
latest backups from amongst all the backup files so that you don't have to
sort through those 1,000+ files. You will need a copy of those latest
backups for the procedure described below.

## Bootstrapping
The aforementioned important `scion_admin` files are housed in SVN. If you 
won't have access to SVN during the server build process (as seems likely),
make sure to have a copy of the `scion_admin` files checked out somewhere.

If you're doing an emergency server rebuild (e.g. due to a catastrophic drive
failure) and you don't have anything but a `scion_admin` backup to work with, 
here's what you can do --

1. Install SVN on the server
1. Create an SVN repos by hand (`svnadmin create /data/svn/temp_scion_admin`)
1. Populate that repos with your most recent backup of `scion_admin`.
1. Run `svnserve` as a temporary SVN server (see http://svnbook.red-bean.com/).
1. Export (`svn export`) the repository into a temporary directory.

*Alternatively* you can also just copy the 'scion_admin' files from your local 
repository to /data/vespa/scion_admin using a safe application such as WinSCP.
In this case you must make sure that all files are owned by root. And that 
certain files (cron.daily.sh, cron.weekly.sh, and post-commit.sh) are executable.

*cron.daily and cron.weekly and post-commit.sh entries must be executable.*
Each file must have the executable bit set (`chmod +x foo`). These files are 
pointed to by a symlink, thus the file to which the symlink points must have the 
executable bit set.

# The Process
## Install Subversion
Install Subversion from the operating system's package manager. On Ubuntu,
you can do this with:
```
apt-get install subversion
```

### Install PySVN
The package [pysvn](http://pysvn.tigris.org/) enables Python access to Subversion. We need it for `svn_cop.py`.

To install:
```
apt-get install python-subversion
```

## Install Trac
The straightforward way to install Trac is from the package manager
(`apt-get install trac`). On the current incarnation of scion, I installed
Trac from source using [their guide](http://trac.edgewall.org/wiki/TracInstall).

I installed from source because Ubuntu's package manager only offered Trac 
0.12.1 and we wanted 0.12.3 (which was the latest stable version at the time
of installation). In addition, we wanted to 
fix the [private:ticket:7 fix the HttpOnly problem] by manually applying
[the HttpOnly patch](http://trac.edgewall.org/ticket/10453). Had we installed
Trac from the package manager, our hand-applied patch might have gotten
overwritten by a package update.

Trac's HttpOnly feature should be included in 0.13. So if you can get Trac >= 0.13 from your package manager, use it. Otherwise, install by hand as I did.

### Install Pygments
Pygments is a syntax colorer that Trac will take advantage of if it's installed.
It's not necessary, but nice. You can install it before or after installing 
Trac.

```
apt-get install python-pygments
```


## Create Directories
```
mkdir /data
mkdir /data/svn
mkdir /data/trac
mkdir /data/vespa
ln -s /data/svn /data/guest_svn
```

The `guest_svn` symlink makes it simpler to configure Apache to allow anonymous
access to SVN.

As mentioned above, the directory `/data/vespa/scion_admin` contains our 
all-important custom scripts. If the SVN server is available to you, check
out scion admin like so:

```
svn co https://your_subversion_server/svn/scion_admin/ /data/vespa/scion_admin
```

In SVN isn't available, create the directory by hand, populate it with the
scion_admin scripts from backups and then turn it into a proper 
SVN-managed directory once you have SVN back up.


## Build SVN Data
### Mailer.py
Get an appropriate version of `mailer.py`. This is a helper script supplied
by Subversion, although it isn't part of a standard install.

To get a copy, figure out what version of SVN is installed 
(`apt-cache showpkg subversion`) and then navigate to that tag here:
http://svn.apache.org/repos/asf/subversion/tags/

From that tag directory, grab `tools/hook-scripts/mailer/mailer.py` and 
copy it to `/data/svn` using something like `wget`. For instance:

```
cd /data/svn
wget http://svn.apache.org/repos/asf/subversion/tags/1.6.17/tools/hook-scripts/mailer/mailer.py	
```


### SVN Repository Infrastructure
There's a scion_admin script called `create_new_svn_repos.py` that simplifies
repos creation. You'll need to run it once for each repository you want to
create. You'll end up executing something like the commands below.

Note 1: The first line creates a convenience symlink. 

Note 2: This is just an example. Get an accurate list of repositories; don't
just blindly copy the commands below.

```
ln -s /data/vespa/scion_admin/create_new_svn_repos.py

python create_new_svn_repos.py contrib
python create_new_svn_repos.py disprior
python create_new_svn_repos.py divox
python create_new_svn_repos.py fetch
python create_new_svn_repos.py fitt_standalone
python create_new_svn_repos.py gamma
python create_new_svn_repos.py gamma_docs
python create_new_svn_repos.py gava
python create_new_svn_repos.py hlsvdpro
python create_new_svn_repos.py ice_python
python create_new_svn_repos.py idealtemp
python create_new_svn_repos.py idl_vespa
python create_new_svn_repos.py libidl
python create_new_svn_repos.py libidl_functor
python create_new_svn_repos.py libidlwave
python create_new_svn_repos.py liblund
python create_new_svn_repos.py mueller
python create_new_svn_repos.py orphans
python create_new_svn_repos.py pywavelets-osx
python create_new_svn_repos.py pyxim
python create_new_svn_repos.py sample_data
python create_new_svn_repos.py sandbox
python create_new_svn_repos.py scion_admin
python create_new_svn_repos.py siemens
python create_new_svn_repos.py sitools
python create_new_svn_repos.py specsock
python create_new_svn_repos.py templates
python create_new_svn_repos.py therm
python create_new_svn_repos.py vespa
python create_new_svn_repos.py vespa_docs
python create_new_svn_repos.py washout
python create_new_svn_repos.py ximo
```

NB. As described in the SubversionPostCommit wiki page, this script creates our svn repositories and sets each one to run certain post-commit actions. This is done by creating a link in the hooks directory to the post-commit.sh file in scion_admin. In migrating to scion2 (July 2018) I found that I needed to make post-commit.sh in the /data/vespa/scion_admin directory executable. 

```
sudo chmod 755 /data/vespa/scion_admin/post-commit.sh
```

Otherwise I got a "Warning: post-commit hook failed (exit code 255) with no output." every time I checked in a repository.

### SVN Repository Data
Look at the backup files that `grab_latest_backups.py` assembled
for you. Move the SVN backups into `/data/svn`.
Leave the Trac backups where they're at for now. 

The SVN backups are 
bzipped. You can un-bzip each one with a command like this:

```
bzip2 --decompress 	svn.disprior.dump.bz2
```

Or you can decompress the whole mess like so:

```
bzip2 --decompress 	*
```

Next, tell svn to restore each repository's data. The commands will look 
something like the ones below. 

This will take a few minutes to execute.

```
svnadmin load disprior < svn.disprior.dump && rm svn.disprior.dump
svnadmin load divox < svn.divox.dump && rm svn.divox.dump
svnadmin load fetch < svn.fetch.dump && rm svn.fetch.dump
svnadmin load gamma < svn.gamma.dump && rm svn.gamma.dump
svnadmin load gamma_docs < svn.gamma_docs.dump && rm svn.gamma_docs.dump
svnadmin load gava < svn.gava.dump && rm svn.gava.dump
svnadmin load idealtemp < svn.idealtemp.dump && rm svn.idealtemp.dump
svnadmin load idl_vespa < svn.idl_vespa.dump && rm svn.idl_vespa.dump
svnadmin load libidl < svn.libidl.dump && rm svn.libidl.dump
svnadmin load libidl_functor < svn.libidl_functor.dump && rm svn.libidl_functor.dump
svnadmin load libidlwave < svn.libidlwave.dump && rm svn.libidlwave.dump
svnadmin load liblund < svn.liblund.dump && rm svn.liblund.dump
svnadmin load mueller < svn.mueller.dump && rm svn.mueller.dump
svnadmin load orphans < svn.orphans.dump && rm svn.orphans.dump
svnadmin load prior < svn.prior.dump && rm svn.prior.dump
svnadmin load pywavelets-osx < svn.pywavelets-osx.dump && rm svn.pywavelets-osx.dump
svnadmin load pyxim < svn.pyxim.dump && rm svn.pyxim.dump
svnadmin load sample_data < svn.sample_data.dump && rm svn.sample_data.dump
svnadmin load sandbox < svn.sandbox.dump && rm svn.sandbox.dump
svnadmin load scion_admin < svn.scion_admin.dump && rm svn.scion_admin.dump
svnadmin load sitools < svn.sitools.dump && rm svn.sitools.dump
svnadmin load specsock < svn.specsock.dump && rm svn.specsock.dump
svnadmin load therm < svn.therm.dump && rm svn.therm.dump
svnadmin load twix < svn.twix.dump && rm svn.twix.dump
svnadmin load vespa < svn.vespa.dump && rm svn.vespa.dump
svnadmin load vespa_docs < svn.vespa_docs.dump && rm svn.vespa_docs.dump
svnadmin load ximo < svn.ximo.dump && rm svn.ximo.dump
```


Apache needs to own all of the SVN data. The name of the Apache user varies 
from system to system. On RHEL it was 'httpd', I think. On Ubuntu it is
'www-data'.

```
chown -R www-data:www-data *
```



## Trac Instance Data
```
cd /data/trac
```

Move the Trac backups into `/data/trac`. They're tarred and bzipped. You need
to untar each one. Sorry, I don't know of a way to tell tar to operate on
every file in a list of directories. You'll have to execute one command for
each, sort of like this:

```
tar xvjf trac.analysis.tar.bz2
tar xvjf trac.contrib.tar.bz2
tar xvjf trac.gamma.tar.bz2
tar xvjf trac.idl_vespa.tar.bz2
tar xvjf trac.private.tar.bz2
tar xvjf trac.project.tar.bz2
tar xvjf trac.rfpulse.tar.bz2
tar xvjf trac.sandbox.tar.bz2
tar xvjf trac.simulation.tar.bz2
```

Apache needs to own all of this, so:

```
chown -R www-data:www-data *
```

If you're changing Trac versions, you need to explicitly upgrade Trac's data and
its wiki data.

```
trac-admin /data/trac/analysis upgrade
trac-admin /data/trac/analysis wiki upgrade
trac-admin /data/trac/contrib upgrade
trac-admin /data/trac/contrib wiki upgrade
trac-admin /data/trac/gamma upgrade
trac-admin /data/trac/gamma wiki upgrade
trac-admin /data/trac/idl_vespa upgrade
trac-admin /data/trac/idl_vespa wiki upgrade
trac-admin /data/trac/private upgrade
trac-admin /data/trac/private wiki upgrade
trac-admin /data/trac/project upgrade
trac-admin /data/trac/project wiki upgrade
trac-admin /data/trac/rfpulse upgrade
trac-admin /data/trac/rfpulse wiki upgrade
trac-admin /data/trac/sandbox upgrade
trac-admin /data/trac/sandbox wiki upgrade
trac-admin /data/trac/simulation upgrade
trac-admin /data/trac/simulation wiki upgrade
```

Last but not least, create a convenience symlink to the script for creating a new Trac instance:
```
ln -s /data/vespa/scion_admin/create_new_trac_instance.py
```

## WWW files
The location of the WWW file directory varies from one operating system to another. On RHEL it was `/var/www/html`, on Ubuntu it is `/var/www`. 

```
cd /var/www
mkdir stats
mkdir awstats_icons
mkdir vespa
```


Copy everything in `scion_admin/www` to `/var/www`. Then move `index.html`
and `vespa_logo2_vertical_small.png` into `vespa`.

```
cp /data/vespa/scion_admin/www/* .

mv index.html vespa
mv vespa_logo2_vertical_small.png vespa
```


## AWStats
AWStats is the package we use to generate Web stats. Most people use it as 
a CGI script. To be extra safe and secure, we don't. We run it daily via a cron job. (See 
`scion_admin/run_awstats.py`). 

*Don't install AwStats via the package manager.* That will set it up to run as a CGI script. 

Instead, grab the latest tarball and unpack it in `/data/vespa/`. Note. as of scion2 in July 2018
I grabbed version 7.7 and adjusted the directions below:

```
cd /data/vespa

wget http://prdownloads.sourceforge.net/awstats/awstats-7.7.tar.gz

tar xzvf awstats-7.7.tar.gz && rm awstats-7.7.tar.gz
```

AWStats can run just fine from that location, so once the tarball is unpacked,
it's already mostly "installed". There's a few things you need to do to complete
the installation.

Note that the location of AWStats is hardcoded in `scion_admin/run_awstats.py`.
Double check that the location in the script matches the directory you just
created. 

Copy the Web graphics:
```
cp -r /data/vespa/awstats-7.7/wwwroot/icon/*  /var/www/awstats_icons
```

Create a work directory:
```
mkdir /var/cache/awstats
```

Create a configuration directory, copy the default config file to it, and 
add our config overrides file:
```
mkdir /etc/awstats
cp /data/vespa/awstats-7.7/wwwroot/cgi-bin/awstats.model.conf /etc/awstats/awstats.scion.duhs.duke.edu.conf
ln -s /data/vespa/scion_admin/awstats.conf.local /etc/awstats/awstats.conf.local
```

Edit `/etc/awstats/awstats.scion.duhs.duke.edu.conf` and add this line at the very end if it's not there already:

```
Include "/etc/awstats/awstats.conf.local"
```

The config file `awstats.conf.local` contains specifics for our environment. You might need to edit this file if you're working in a different context (different OS, Apache version, or AWStats version). The default config file (`awstats.scion.duhs.duke.edu.conf`) that AWStats supplies is very well commented.

## (Deprecated for scion2 July 2018 - Patch awstats.pl?
We're using AWStats 7.0 (build 1.971). That version contains
[a bug exposed under Perl >= 5.14](http://sourceforge.net/tracker/index.php?func=detail&aid=3311848&group_id=13764&atid=113764). The consequence of this bug for us was that AWStats' keyword and keyphrase reports were always empty. Based on the conversations I read on sourceforge, this bug breaks other things in AWStats' reports in addition to what I observed.

This bug is reported to be fixed in [version 7.1 as of October 2012](http://sourceforge.net/p/awstats/bugs/868/?limit=10&page=1#1fd8) so hopefully you can ignore this. If it's not fixed, you can fix it yourself pretty easily. Simply apply the file `scion_admin/awstats.patch` to `awstats.pl`:

```
cd /data/vespa/awstats-7.0/wwwroot/cgi-bin
mv awstats.pl awstats.pl.original
cp awstats.pl.original awstats.pl 
patch < /data/vespa/scion_admin/awstats.patch
```

If this fails for whatever reason, just look at the patch file yourself. (It's just ASCII output from diff.) You can apply the changes by hand in about 60 seconds.


## Apache Config
Apache offers lots of pluggable modules, and it is a little hard to predict 
which will be installed and/or available on the machine on which you're 
working. Also, different operating systems/Linux distros do things differently.
I'll give you instructions that worked for me on Ubuntu, but you might have to
do a little work here to figure out what to do on your platform.


Install mod-python and the SVN module (which provides mod_dav_svn and 
mod_authz_svn).

```
apt-get install libapache2-mod-python
apt-get install libapache2-svn 
```

Enable mod_rewrite and authz_dbm. Also, we need neither PHP nor CGI, so I 
disabled them both:
```
cd /etc/apache2/mods-enabled
ln -s ../mods-available/authz_dbm.load
ln -s ../mods-available/rewrite.load
rm cgi.load
rm php5.load
rm php5.conf
```


Modify `/etc/apache2/sites-enabled/000-default` and add this to 
`<VirtualHost *:80>`:
```
RewriteEngine On
RewriteOptions Inherit
```

Why? See this: https://bugs.launchpad.net/ubuntu/+source/apache2/+bug/820936

Restore `/etc/apache2/.htpasswd` and `/etc/apache2/auth_groups.*` from backups.

Link to Vespa's config files:
```
cd /etc/apache2/conf.d
ln -s /data/vespa/scion_admin/apache_conf/subversion.conf
ln -s /data/vespa/scion_admin/apache_conf/vespa.conf
```

NB. On scion2 (newer Ubuntu) conf.d no longer exists. Rather conf-available contains 
available configuration files. All files that were previously in /etc/apache2/conf.d 
should be moved to /etc/apache2/conf-available and we need to link from conf-enabled
to the files in conf-available then ones that we want to be applied.

Links in conf-avalilable to Vespa's config files:
```
cd /etc/apache2/conf-available
ln -s /data/vespa/scion_admin/apache_conf/subversion.conf
ln -s /data/vespa/scion_admin/apache_conf/vespa.conf
```

Links in conf-enabled to conf-available to turn them on:
```
cd /etc/apache2/conf-enabled 
ln -s /etc/apache2/conf-available/subversion.conf
ln -s /etc/apache2/conf-available/vespa.conf
```


Restart apache.

```
apachectl graceful
```


## Cron
Our Ubuntu had [anacron](http://en.wikipedia.org/wiki/Anacron) enabled by
default. We (both Kim Greer and I) found this irritating and disabled it. This
was a two-step process. First, I made anacron not executable with this command:
```
chmod -x /usr/sbin/anacron
```

Several default Ubuntu files (`/etc/crontab`, `/etc/cron.daily/0anacron`)
check whether or not `/usr/sbin/anacron` is executable, so I think that making
it not executable is the preferred method of disabling it.
[Others agree](http://lists.clug.org.za/pipermail/clug-tech/2006-December/035207.html).

In addition, I edited `/etc/crontab` and removed all references to anacron.

I also changed cron.daily to run at 1AM and cron.weekly at 2AM. Most of our
users outside of North America are in Europe, so I chose to run our cron jobs
at times when the North Americans and Europeans would likely be sleeping 
so as not to load the server when people are trying to use it. (Sorry, Asians.)
In our case the cron jobs are not too demanding so it doesn't matter much,
but this is good practice anyway.

I tried to minimize my changes to `/etc/crontab`. 
[If it gets overwritten in some future update](http://ubuntuforums.org/showthread.php?t=1969596), 
no harm will be done except that our cron jobs will run
at a different time than we expect.

We have scripts for cron.daily and cron.weekly. They're in `scion_admin` and
called (surprisly enough) `cron.daily.sh` and `cron.weekly.sh`. All you have to 
do is symlink to them:
```
ln -s /data/vespa/scion_admin/cron.daily.sh     /etc/cron.daily/vespa
ln -s /data/vespa/scion_admin/cron.weekly.sh    /etc/cron.weekly/vespa
```

## Remote Backup Configuration
As of December 2012, scion copies its backups to an account on another machine 
at Duke. Currently that is `bsoher@spect5.duhs.duke.edu`.

For the backup scripts to work correctly, `root@scion` needs to be able to log in to the remote account without being prompted for a password. We do this by copying SSH keys. Here's how.

While logged in to scion as root, run `ssh-keygen`. Accept all defaults and don't enter a password. That will create the file `/root/.ssh/id_rsa.pub`. 

The file `id_rsa.pub` needs to get to the remote user's account. There are two ways to do this. One is with this command --
```
ssh-copy-id -i ~/.ssh/id_rsa.pub bsoher@spect5.duhs.duke.edu
```

There's also the manual way which I prefer because it is less magic, although it requires more steps --

1. Copy `/root/.ssh/id_rsa.pub` to the remote account --
```
scp /root/.ssh/id_rsa.pub  bsoher@spect5.duhs.duke.edu:~
```
1. Log in to the remote account.
1. Create the directory `.ssh` if it doesn't already exist.
1. Append `id_rsa.pub` to `~/.ssh/authorized_keys`. The command below will work regardless of whether or not that file already exists. (There's a good chance it won't).
```
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
```
1. Clean up --
```
rm ~/id_rsa.pub
```
1. Log out of the remote server.

Whether you use `ssh-copy-id` or the manual method, you should test to see that `root@scion` can ssh to `bsoher@spect5.duhs.duke.edu` without a password.

## Troubleshooting
Here's a few subtle mistakes that are easy to make.

*The Apache user must have write access to /data/trac and /data/svn.* It's 
easy to end up with some or all of that owned by root in which case Trac and
SVN will report strange (unintuitive) errors.

You can either make all of that data writeable by everyone (bad idea!) or make
the Apache user the owner of it:
```
cd /data/svn
chown -R www-data:www-data *
cd /data/trac
chown -R www-data:www-data *
```

Interestingly enough, it doesn't matter who owns the files in `/var/www` as
long as Apache can read them. 


*On Ubuntu, cron.daily and cron.weekly filenames are ignored if they contain a dot.*
It's a feature. (From 
[the run-parts man page](http://manpages.ubuntu.com/manpages/precise/en/man8/run-parts.8.html): 
"run-parts runs all the executable files named within constraints described
below, found in directory directory. Other files and directories are silently
ignored".  So, `run_foo.sh` will be ignored, `run_foo` will be executed.

*cron.daily and cron.weekly entries must be executable.*
Each file must have the executable bit set (`chmod +x foo`). The the entry is
a symlink, the file to which the symlink points must have the executable bit
set.
