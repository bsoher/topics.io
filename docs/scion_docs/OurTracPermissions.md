---
sort: 14
---

# Our Trac Permissions

## Users and Permissions
[UsersPasswordsAndPermissions Apache handles authentication for Trac] 
(just like Apache handles authentication for Subversion).

Permissions (answering the question of who has access to what) are handled
by Trac with groups. Trac has built-in groups of "authenticated" (logged-in) and  "anonymous" (not logged-in). To this we've added two custom groups: "editor" and "admin". 

*The editor group* sounded good when we created it, but we haven't actually used it much. The idea was to give write access to non-core members of the project without giving them delete access. It turns out that we haven't had anyone in that role, so the editor group doesn't serve much purpose.

*The admin group* is what is sounds like: diety powers. In practice (since the editor group isn't used), we admins are the only one who log into Trac, so the admin group could just be represented by the *authenticated* group.

The powers granted to the *anonymous* group varies. For examples, in the "private" Trac instance (this one), anonymous users have no privileges. But in the "contrib" Trac instance, anonymous users have read privileges.

## Editing Group Membership and Privileges
One can edit all of this stuff by clicking on the Admin menu bar item on any Trac instance. (You have to be logged in.)  For details about permissions, refer to the 
[TracPermissions Trac permissions guide]. (It's not difficult.)

## Defaults for New Instances
By default, there's no one in the `editor` group of a new Trac instance and
the `admin` group is restricted to me (Philip), Brian, and Karl.
The `create_create_new_trac_instance.py` script controls that setup.

## Comments on Putting Anonymous Access into a Wiki
*NB. bjs As of 2018-07-27 for scion2 upgrade, I've added the anonymous group and permission back into the create_new_trac_instance.py script. This seems to work OK despite the comment below. I know that the script was previously removing all anonymous permissions without putting them back in, so maybe that was the issue.  Anyway ... this seems to work now as part of setup*

I've found that sometimes (most times?) the create_new_trac_instance.py does not include permissions/authentications for anonymous access.  When this happens, I go onto scion, become super-user and do it by hand. I use the command below to add the various permissions for group anonymous:

`>root@scion:/data/trac# trac-admin hlsvdpro permission add anonymous ROADMAP_VIEW`

I repeat this for: BROWSER_VIEW  CHANGESET_VIEW  FILE_VIEW  LOG_VIEW  MILESTONE_VIEW  REPORT_SQL_VIEW  REPORT_VIEW  ROADMAP_VIEW  SEARCH_VIEW  TICKET_VIEW  TIMELINE_VIEW  WIKI_VIEW

Once these are all typoed in, the wiki seems to behave properly.
