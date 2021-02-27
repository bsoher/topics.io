---
sort: 11
---

# AWStats on Scion

[AWStats](http://awstats.sourceforge.net/) reads Apache's access log and
generates stats about the visitors to our Web site. This document explains
a little about our configuration, how it runs, etc.

The reports it generates are here: https://scion.duhs.duke.edu/stats/

The Python script I wrote for this and the AWStats config file we use are
in the `scion_admin` SVN repos.

## The Config on Scion
AWStats can be run on a Web server via cgi-bin, and I think that's very
popular on large hosting services because it allows Webmasters to fiddle 
with AWStats' parameters and then re-run the stats as they please. AWStats'
documentation is very much oriented to cgi-bin invocation.

We're using it rather differently. I do _not_ want AWStats available via 
cgi-bin on scion because we don't need up-to-the-minute stats, and 
having it available via cgi-bin could pose a security risk if vulnerabilities 
are found in AWStats.
Instead, I run AWStats from the command line in a daily cron job.

The [ScionHowToBuild scion build document] explains how we install AWStats and
I won't repeat that information here. I'll just note that since our setup is 
atypical, setting up AWStats takes a little more work than a typical setup 
would.

## Running from the Command Line/Cron
The daily `cron` script runs AWStats automatically. If you want to run it
manually, that's no problem. (It won't confuse the `cron` script.) Just 
run `scion_admin/run_awstats.py`.

That Python script is meant to be run every day. Nothing bad will
happen if you skip a day or three, but you won't get accurate stats for 
the previous month if `run_awstats.py` fails to run on the first day of a 
new month.


## Password Protection
I chose to require authentication to access the reports that AWStats 
generates because the log data might reveal things we'd rather not 
make public, like the existence of certain SVN repositories or pages inside 
the private Trac instance.

Access is restricted to the users hardcoded in `/etc/httpd.d/awstats.conf`.
The password is the same as for SVN and Trac. Since passwords are required, 
access is restricted to HTTPS (no HTTP) to avoid leaking passwords on 
public networks.


