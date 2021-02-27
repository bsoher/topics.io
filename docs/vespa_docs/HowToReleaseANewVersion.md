---
sort: 1
---

# How to Release a New Vespa Version

## Step-by-Step

Here's the steps to take when you're ready to make a new version of Vespa available to the world. In the examples below, we'll pretend we're releasing Vespa 0.1.4.

If you're on Windows, you will first need to download  [tar for Windows](http://gnuwin32.sourceforge.net/packages/gtar.htm) and [gzip for Windows](http://gnuwin32.sourceforge.net/packages/gzip.htm). They need to be in your `PATH` before you run step 5 (`setup.py sdist`).

Also, on Windows you need to use Python > 2.7.1 to build the bundles. If you use an older Python, the executable bits on the directories won't be set in the tarball and non-Windows users won't be able to browse it. (Sorry, I can't find a ticket in the Python bug tracker for this.)



1. Announce a code freeze to your co-developers. That means sending out an
email saying, "Vespa trunk is currently at revision 54321 and we're 
freezing it there for version 0.1.4."

2. Update the VERSION and NEWS files if you haven't already. You might want
to read my tutorial on how I [HowToWriteTheNews write the NEWS file].

3. Commit your changes to SVN.

4. Create a tag for the frozen version. If you're creating a tag based
on HEAD, use a command like this (replacing the stuff in caps) -- 
  
    ```
      svn copy https://scion.duhs.duke.edu/svn/REPOS_NAME/trunk         \
               https://scion.duhs.duke.edu/svn/REPOS_NAME/tags/TAG_NAME \
               -m "TAG COMMENT GOES HERE."
    ```
  
    For example --

    ```
      svn copy https://scion.duhs.duke.edu/svn/sandbox/trunk             \
               https://scion.duhs.duke.edu/svn/sandbox/tags/practice_tag \
               -m "Creating a tag for funs and giggles."
    ```

    For the release of Vespa version 0.1.4 --
  
    ```
      svn copy https://scion.duhs.duke.edu/svn/vespa/trunk             \
               https://scion.duhs.duke.edu/svn/vespa/tags/0_1_4        \
               -m "Tag for Vespa version 0.1.4."
    ```
  
    I've never needed to do this for a release, but if you want to create a 
    tag from a revision other than HEAD, just pass the revision number to 
    svn copy -- 
  
    ```
      svn copy -r 400                                                  \
             https://scion.duhs.duke.edu/svn/sandbox/trunk             \
             https://scion.duhs.duke.edu/svn/sandbox/tags/tag_400      \
          -m "Creating a tag for revision 400."
    ```

5. Create the bundles by running this --  

    ```
      python setup.py sdist --formats=gztar,zip
    ```

    That creates `dist/Vespa-0.1.4.tar.gz` and `dist/Vespa-0.1.4.zip` along with `.md5` and `.sha1` hash files of each. 
  
6. Attach each of the six files to [project:wiki:AllDownloads the AllDownloads page].

7. Update the links on the [project:wiki:Downloads the Downloads page]. There's a lot of references to the version number on that page, but there's an easy way to make sure you change them all. Edit the page, copy/paste all of the wiki-formatted text into a text editor,  do a global search/replace from the old version number to the new, paste the updated text back into your browser.

8. Announce the new version on the Vespa email list. 
