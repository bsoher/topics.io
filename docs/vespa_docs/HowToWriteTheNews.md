---
sort: 3
---

# How to Write the NEWS File

Each Vespa release goes out with a _NEWS_ file that highlights the important changes in the release. What goes into the NEWS file forms the core of the release message that's sent to the Vespa MRS mailing list. Here's how I figure out what to put in the NEWS file.

In the instructions below, I assume I'm releasing Vespa 1.2.4 and that the previous version was 1.2.3. I'm also assuming that I'm on Windows and using TortoiseGit (since that is what I know).

1. Go to the source code repository in Windows Explorer.
1. Right-click on TortoiseGit->Tags and note the revision that tagged the previous version. In my example, tag 1_2_3 was commited in revision 2408. Note the date, too.
1. Right-click on TortoiseGit->Show log and a dialog will pop up. 
1. Select a date range from the last tag til now.
1. This will show a list of Messages included in each commit.  
1. Look through those messages, pick out the highlights, and there's your NEWS!


