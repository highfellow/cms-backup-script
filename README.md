CMS Backup script
=================

This is a script which handles backups of a content management system. I.e. some files plus one or more databases. The files can be specified quite flexibly using regular expressions which define paths to include or exclude relative to a given top level directory. The databases can be (currently) either mysql or postgresql. All of the backed up files and databases are wrapped into a single tar file, whose name gives the name of the installation being backed up, plus a date stamp.

It can be invoked in either manual or scheduled mode. Manual mode makes a single backup with a date stamp; scheduled mode can be called from a cron job to generate a series of nightly, weekly, and monthly backups.

This is now working, though may have some bugs. 

To install, clone the repo and symlink `backup` to `/usr/local/bin/backup`, and `backup.ini` to `/usr/local/etc/backup.ini`. Make a directory `backup.d` in `/usr/local/etc`; this contains any number of `<backup-config-name>.ini` files, which define a particular set of files and databases to back up. `test.ini` in this directory can be used as a template.
