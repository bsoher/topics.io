---
sort: 4
---

# Occasional Tasks

There are a few administrative jobs that need doing "every so often".  Some can happen on a schedule, some don't.

## Weekly

1. Read the backup status emails. This isn't too hard to remember since you get a push notification in the form of a status email in your inbox.

## Monthly

1. Check the Web stats. The data are most useful in aggregate and it's not that exciting anyway, so having a look at the previous month's stats at the beginning of each month is about the right frequency. There's no automatic notifications for this. 

2. Consider [updating the Vespa appliance](VespaAppliance.md). Even if Vespa doesn't change, the OS & tools will have updates applied. It's a nicer user experience if the users don't have to deal with this.

## Annual

1. Review account and permissions (both SVN and Trac) to see if there's any moribund accounts that should have access scaled back or be removed entirely.

## Now and Then

1. Check to see if any of Vespa's dependencies (e.g. numpy) have come out with new versions. Ideally, we would always recommend the latest and greatest on Installation our installation page but sometimes it isn't possible (e.g. when PyWavelets didn't support Python 2.7 on Windows). 

1. Check the status of wxPython 2.9. This is sort of a special case of the above. It might be a while before Python 2.9 is ready for prime time. 

1. Check on [nmrglue's](http://code.google.com/p/nmrglue) development. Remember that we copied some pieces of nmrglue into our code, so as improvements are applied to the nmrglue code, it will be useful to reflect them in Vespa too.