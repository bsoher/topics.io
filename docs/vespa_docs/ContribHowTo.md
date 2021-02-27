---
sort: 7
---

# How to Contribute to Vespa

## How To Create a Contributions Page

A contributions page is just an ordinary Wiki page. You can write it entirely by 
hand if you want, but writing each one by hand would be a lot of work
and result in inconsistent formatting. We can solve both of those 
problems by letting scripts do most of the work. 

There are two helper scripts. 

They're meant to be run one after another. They massage the input into the
output with several opportunities for a human (that's you!) to intervene if
necessary. The scripts' input, intermediate and output files are all plain 
text to make them easy to manipulate.


## Assumptions

* You have the contrib code (Python scripts and templates) in a directory
called `contrib`. This can be on your PC, we don't need to be on scion.

* A contributor has filled out our questionnaire and that you have her answer
saved as `contrib/work_area/answer.txt`.

* Any contributed supporting files (VIFFs, etc.) are also saved in 
`contrib/work_area`.


## 1 - Parse the answer

Parse the anwser with `parse.py`. 

This script takes as input the name
of the file containing the contributor's reply (`answer.txt`) and all 
files that are to be used as attachments. For example, from inside the work_area directory --

```
python parse.py answer.txt my_thesis.pdf
```

The parser extracts the contributor's questionnaire answers. If any of the
attachments are VIFF files, the parser will extract data from them too.
Next it reformats the data and writes 
several plain text files with names like `author.txt`, 
`license.txt`, `short_description.txt`, etc.

These files are the _intermediate files_.

## 2 - (Optional) Touch Up the Intermediate Files

The intermediate files contain Trac Wiki markup. Ideally, you won't 
need to edit them, but this process is deliberately structured so that 
you _can_ edit them if you need to.

For instance, if the contributor sent a reply embedded in a "rich text" email 
that contained italic and bold formatting in the long description, that 
formatting will be lost when you save it as plain text for step 1's input. 
This is your chance to open `long_description.txt` in a text editor and 
reapply the user's formatting using Trac Wiki syntax.

## 3 - Generate the Page

Generate the page with `generate_page.py`. 

This script takes no input and writes `UUID.txt` as its output, where
UUID is a randomly-generated UUID for this page. The content
of the output file is ready to be pasted into the Wiki.

This program is very simple; it reads `template.txt` and inserts the content
of the filenames produced by `parser.py` into the template.

## 4 - Create the Wiki Page

With your browser, create a Wiki page using the UUID generated in step 3 as 
the name. For example --

```
https://scion.duhs.duke.edu/vespa/contrib/wiki/602e7450-8227-48eb-8252-38f8597d9342
```

To create the content of that page, paste in the entire contents of `UUID.txt`.

- Note 1. Through some magic Philip got rid of the metanav bar on this Trac wiki, so I am no longer given the option to log in. 
- Note 2. I typed https://scion.duhs.duke.edu/vespa/contrib/wiki/login" and it let me login/edit pages
- Note 3. Orig instructions for creating a new page were not working so now I go to the appropriate sub-page (eg. Pulse Sequences) and add a link to the new page (based on the UUID) I want to make. Save changes and go to that link. When I click on it Trac offers to create it and allows me to edit it.

## 5 - Upload the Attachments

Each page references zero or more attachments. You need to add them to the 
Wiki page manually using the "Attach File" button at the bottom of the Wiki
page.

## 6 - Review

_"To err is human, but to really screw up requires a computer."_

*You are ultimately responsible for the content of the page.* 
These scripts are assistants that guess intelligently, but computers
can be awfully naive. Double check your work.

## 7 - Add/Update References

Each submission should be referenced from the appropriate TOC page
(https://scion.duhs.duke.edu/vespa/contrib/wiki/PulseProjects,
https://scion.duhs.duke.edu/vespa/contrib/wiki/Experiments, etc.). Add an entry
there.

## 8 - Clean Up

When you're done, clean up your `contrib/work_area` directory by 
deleting everything but `parse.py` and `generate_page.py`.

## About the Page Template

The page template (`template.txt`) is just a Trac Wiki page with some 
additional magic. The magic is in several strings delimited by dollar signs
(e.g. `$license.txt$`). The text between the dollar signs must be a file name.
The script `generate_page.py` replaces the text and dollar signs by the 
content of the named file. I chose $ as the delimiter because it seems 
unlikely to be used elsewhere in the template.



# History of the Contributions Page - Or, how we messed it all up

## A Warehouse for Contributions

This document contains my suggestions for how we might organize a Trac
instance for Vespa contributions. These contributions would consist of
exported experiments, metabolites, pulse sequences, pulse projects 
authored by Vespa users.
*I suggest we also accept contributions of PyGAMMA scripts.*

  bjs - I concur on all contributions

## Dedicated Trac Instance

*I suggest that we create a dedicated Trac instance for the contributions.*

We discussed whether or not contributions should go into the existing
Vespa (project) Trac instance or into a new, dedicated Trac instance. A dedicated 
instance has the following advantages --
 * Makes searching more effective since there won't be any Vespa clutter to 
 wade through
 * Gives us more flexibility with permissions.
 * Allows us to customize the Wiki for its specialized purpose

The only disadvantage is that it's another Trac instance to manage, but we'll
barely notice.

If we think we're going to get flooded with contributions, we might consider
a separate Trac instance for each category of contribution (metabs, experiments,
etc.) but that seems probably too fine-grained.

  bjs - one Trac should be sufficient for anticipated traffic. 

## Wiki Organization

At the finest level, *I suggest we use the UUID of the object as the URL*.
For instance, if I submit GPC-pcholine-philip that has a UUID of 
`0a8d7945-fe26-4a88-a1f4-3197ec85c046`, then the URL to my contribution 
would be --
```
http://scion.duhs.duke.edu/vespa/contrib/0a8d7945-fe26-4a88-a1f4-3197ec85c046
```

UUIDs are ugly and very difficult for a human to type without making a
mistake. But they give us a simple, unambiguous rule for generating URLs.

  bjs - I like the idea of UUIDs as URLs. When a user downloads an xml file to import, they won't have to type it, right? Just click on the link on the wiki?  

  ps - Right. The only reason I mentioned typing the URLs is because in general we never have to type out URLs but it happens from time to time. For example you might see a reference to the URL printed on actual dead trees in which case you're forced to type in the whole thing. So, typability matters once in a while.

*I don't know what to suggest for PyGAMMA script URLs.*

  bjs - to keep it in the same form as the other modules we'll have to assign it a UUID when we put it on the wiki (as below)

  ps - Strictly speaking we don't _have_ to, but it's a good idea.

  bjs - this sounds resovled? Comment only on any changes you want to make to this idea, please.


*I suggest we have an index page for each object type.* In other words,
/Metabolites, /Experiments, /PulseSequences, and /PulseProjects, and 
/PyGAMMA.

  bjs - concur

If we want, we can also make custom index pages. For instance, frequent
contributors might get their own page (e.g. /BrianSoher).

  bjs - maybe someday ...

## Adding Content

*I suggest that we accept submissions only via email.*

Given that we have a Web site and we're asking people to submit information
that will eventually become web pages, one obvious path is to avoid any
middlemen and have a form that people fill out which populates a Wiki page.

However, if we allow anyone to add content, then
spammers will fill the wiki with viagra ads, so submissions have to be
moderated -- there's no avoiding that work. In addition, a Web form isn't
a standard part of Trac (not the Trac version we're using, anyway). Lastly,
processing the form input would involve server-side scrpiting.

Instead I suggest that we develop a plain text form that we send to would-be
contributors. It would ask them to provide their name, organizational
affiliation, type of contribution, citation information, etc. It could
certainly allow or even encourage free-form response in certain sections, but
a question format will help to ensure we get the information we need.

If we design the form carefully, we can also attempt some automated processing
on the responses. We could write a script that would run on your machine
that would take a filled-in response form and convert it into TracWiki syntax.
Some hand-editing would still be necessary, but it would hopefully 
obviate some of the grunt work.

  bjs - would it be possible to make a button on the wiki that pops up a mail client pre-populated with the form? I'd like to make this process a bit brainless as long as it doesn't get too hard to do.  I like the thought of some automation of the parsing of the emails.

 ps - No, it's not possible to pre-populate emails. We ran into this problem when we were designing the Vespa bug reporter, remember? And as David pointed out then, some people don't even have an email client. They just use gmail or yahoo or somesuch. That said, we can make available a text file that contributors can open and paste into the body of an email. It's a few more steps, but 100% reliable and very little work for either party.

  bjs - fair enough, I should have recalled this issue ... simple text form it is.

## Server-Side Scripting

*I recommend we avoid using much, if any, server-side scripting due to the maintenance time and expertise required.*

  bjs - I'm ok with anything that makes future management of the website simpler.

  ps - "simpler" is a very ambiguous term here. Using server-side scripting would make maintenance both more and less simple. It's the classic tradeoff that comes with almost any kind of automation. 
  It would make life more simple because when e.g. Dr. Alfred E. Neuman submits a metabolite and you add it to the wiki, the script would automatically notice it and add it to the index of metabs (at /Metabolites) and maybe also to /Authors/AlfredENeuman and perhaps /Metabolites/31P and /Metabolites/4spin. Nifty!
  
  It would make life less simple because when the script breaks (as it will inevitably do at some point due to unexpected input, a Trac upgrade, thunderstorm, etc.) then (a) you might not notice and (b) when you go to fix it, you have all of the disadvantages listed below.

  bjs - a 'semanchuk special' break? Heavens forfend ...  Seriously, I'm ok with less scripting than more.



Trac offers the ability to write server-side scripts in Python and have
those scripts generate Web pages. We use a handful of these already, but 
they perform non-essential tasks. (One example is the 
[private:wiki:SubversionRepositories live list of SVN repositories].)

Since the contributions Trac Wiki will have a lot of semi-homogenous content,
it represents an opportunity to leverage scripts to handle some of the
drudgery. For instance, we could have a script that hourly scans for 
new pages. If it finds any, it could parse the attached VIFF file and index
the content. A Wiki page like /Metabolites, then, could be autogenerated
from the database that this script builds.

  bjs - I like this idea. It's basically going to automate updating the various table of content pages, right?

  ps - yes. 




The *advantages* to using a server-side script in this case and in general
are --
 * Handles some repetitive tasks so that you don't have to.
 * We might forget to add a newly-contributed metabolite to the 
   index; a script won't.
 * The usefulness of a tweak or addition to the script is multiplied by the
   number of items it affects. By contrast, hand-editing pages always happens
   just one page at a time. If and when we have, say, 100 items contributed, 
   a change to the script can improve the display or categorization of all 
   of them. Changing all 100 by hand would probably be too much work to 
   consider.

The *disadvantages* are --
 * Trac scripts are written in Python but debugging them can be a little 
 tricky. I'm the only one who has experience with this, and my days on the
 project are numbered. It would be a little scary to have fundamental parts of 
 the wiki rely on scripts that present a maintenance headache. 
 * Automated scripts make it more difficult to customize pages.


## Licensing

What license will apply to contributions? 

*I suggest one of two approaches.* One is to require everyone to use one
license of our choosing (probably our BSD license). An alternative is to 
allow contributors to choose from a predfined list.

  bjs - this is an area that can get complicated, but in some ways is also pretty easy. Most researchers don't care. So, I'd rather err on the side of "easier for us" and just ask them all to accede to the BSD license we use.

  ps - If most researchers don't care about licensing, I suspect it's because they don't pay attention to them much (even when they should). But OK.

In either case, we'll probably have to make exceptions for contributors who
are bound by their organziation to use a certain license. We will also add
public domain to the list, since work from federal government employees 
in many cases must be public domain.

  bjs - I'm a bit afraid that if we allow too much (or any) mix/match of licenses, we will scare off folks who just want to get something and try it.  I'd rather that everything donated to the site be working to facilitate those folks.  If others have cool and fancy modules to share, they can advertise them on the email list and do some direct marketing. I don't want to be the traffic cop (implied or otherwise) for people with difficult licensure issues.

If we allow contributors to choose from a predfined list, I suggest 
[the six Creative Commons licenses](http://creativecommons.org/licenses/?lang=en).

  bjs - can we just do it under our BSD?

  ps - Yes, but...if you want to require that all contributions become BSD licensed without exception, that's your prerogative. I understand that you feel that that makes life easiest for users of the site, and for the most part I agree. However, it might be putting the cart before the horse. If you don't get contributions to the site, then how the site's visitors respond to the content will be a moot point. I made my licensing recommendations with that in mind. I hope that some flexibility in license choice (without presenting an overwhelming list of options) would encourage contributions. A rule stating, "You must accept the BSD license" would discourage some contributors.

  You might, in fact, be forced to turn away some contributors. If someone works for the feds and is not permitted to apply a license to his work, will you refuse his contribution? If someone works for MIT and MIT insists that she use the MIT license http://en.wikipedia.org/wiki/MIT_License ), will you refuse her contribution?

  Also, you say that you feel that a mix of licenses will scare off those who want to get something and try it. Public domain has no restrictions whatsoever and is therefore even less scary than BSD. Not that BSD is scary if one has any experience with it, but it is nevertheless a wad of legalese. That's one of the things I like the Creative Commons licenses. They make a point of presenting their licenses in plain language. There's a legalese version, but it's secondary. Honestly, look at the two links below with a fresh eye. Pretend you've never seen either before and ask yourself which easier to understand. For bonus points, repeat the exercise while pretending that English is not your native language. 
  
  http://creativecommons.org/licenses/by/3.0/
  
  http://scion.duhs.duke.edu/vespa/project/browser/trunk/LICENSE

  Note that the CC licenses are available in over two dozen languages.

  Last but not least, IMO you have nothing to fear regarding a role as a license cop. You'll only find yourself in that role if you put yourself there. If person A contributes to your site and person B misuses that contribution, it makes no sense for A to pressure you to resolve the situation. 

  bjs - if this is nothing more than a line in the wiki and maybe a link to whatever is online about that license, then OK.  But, if we have to add anything to the database to track the licensure or do ANY other coding other than listing it in our wiki prominently, then part of me feels that it is (potentially) too much trouble.  Yes, an easier licensure is better and one that reads well is a boon to the user, but, I don't want to get caught up in the "well, you list these 6 licenses, so why can't I have this 7th one?" issue.  See 'home grown' below.


    ps - Yes, my concept is that each contribution will have a line that says "This contribution is licensed under the Foo license (link)". I agree that we should aim for a low tech system. The link between a contribution and its license will be on that contribution's page and nowhere else. BTW you say, "if we have to add anything to the database..." and that makes me wonder what database you're thinking of. Other than the standard, hidden Trac database that we don't think about or use directly, we're not going to have a database for this Trac instance. Does that fit your perception of what this will be?

  I agree that if we offer six licences it's perhaps a little difficult to say no to adding licenses 7, 8, etc. I just think that if contributors care about licenses at all (no guarantee that they will), then a one-BSD-size-fits-all approach is not going to prove acceptable. 

  I would be willing to go with one-BSD-size-fits-all and have a plan B ready in case that causes problems. However I also see a practical problem with the BSD license. The very first line is a statement that assigns copyright to a specific entity. (For our license: "Copyright (c) 2010, Duke University and the United States Department of Veterans Affairs.") If we take that line out, we're in homegrown license territory and neither one of us likes that idea. If we leave it in, we have to modify it for every contributor. By contrast, the CC license simply say, "You must attribute the work in the manner specified by the author...". One of the things that we'll require of contributors is an answer to the question, "How should people cite your contribution?" and that dovetails nicely with the CC license language.

  One last thing to consider (sorry) is that I just noticed that all six of the CC licenes require authors of derivative works to attribute the author. This seems reasonable to me (esp. for research and academia) but since your top choice (BSD) does not, I thought the difference was worth mentioning.



The main thing to avoid are "one-off" or "homegrown" licenses. Those are
licenses written by non-lawyers that are only used in one place (all works
by a particular author, for instance). They're often ambiguous and therefore
of questionable legal standing.

  bjs - concur


## Document Formats

*For documents, I suggest we accept only non-proprietary format such as PDF, HTML, PNG, JPG, etc.*

  bjs - documents? what documents?  I'm assuming that you are suggesting any files other than the xml export files, or *.py PyGAMMA files?  Do we  want to encourage folks to zip up all relevant files (or gzip) and then we just let them download one big file?

  ps - I'm not sure how this will work, but my guess is that some people are going to submit extra materials along with their contribution. One of the things we'll ask them is to explain what's interesting about their contribution, and what with pictures being worth 1000 words and all that, I'm sure you'll have people sending images & docs along with their text. e.g. "This pulse sequence normalizes the potrzebie factor resulting in a smoother, creamier pulse. Attached are some PNGs from Vespa and a paper I wrote describing the heretofore unknown potrzebie factor."

  bjs - part of me wants to insist that they load all docs into a zip/gzip/rar file so we only have to deal with one file on our wiki-side.

  ps - That's a good point. I don't think it's much work for us no matter what they send -- we attach their files to the relevant page and never have to touch them again. Doesn't much matter if it's one file or ten. But as a user browsing the wiki, would you rather see the attachments listed one by one so it's easy to pick and choose, or all in a big bundle so you can download them all at once? To me, that's the main question. 


I think it's important to avoid proprietary formats (the obvious example
being Microsoft Office documents) for a couple of reasons. First of all, 
they can't be rendered properly by everyone (especially on Linux which covers 
a significant part of Vespa's audience). Second of all, they're a vector for 
viruses and other malware.

  bjs - concur, standard file formats only.
