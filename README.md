CMS Backup script
=================

This is a script which handles backups of a content management system. I.e. some files plus one or more databases. The files can be specified quite flexibly as paths to include or exclude relative to a top level directory given in the global config file. The databases can be (currently) either mysql or postgresql.

Each backup consists of a directory containing all of the backed up files and databases. The directory name gives the name of the backup configuration being run, plus a date stamp. Identical files between backups are hard links to the same inode; this means that each successive backup only uses the extra disk space taken up by the changed or new files. (You need to run the script on a filesystem that supports hard links).

It can be invoked in either manual or automatic mode. Manual mode makes a single backup with a date stamp; automatic mode can be called from a cron job to generate a series of nightly, weekly, and monthly backups.

Installation
------------
To install, clone the repo with `git clone --recursive https://github.com/highfellow/cms-backup-script.git` and symlink `backup` to `/usr/local/bin/backup`, and `backup.ini` to `/usr/local/etc/backup.ini`. Make a directory `backup.d` in `/usr/local/etc`; this contains any number of `<configname>.ini` files, which define a particular set of files and databases to back up. `example.ini` in this directory can be used as a template. For more information, see the comments in `backup.ini` and `example.ini`.

Invocation
----------
You invoke the script as `backup [-a] <configname> ...`. (Note more than one config is possible.) E.g. `backup -a myblog clientsite` to take a nightly backup of your personal blog and a site you have done for a client.

If there is no `-a` you are in manual mode, and the backup(s) will end up in `<backupdir>/<configname>/manual` where `<backupdir>` is the backup directory you specified in the global config file.

With the `-a`, you are in automatic mode, which is intended to be used from a nightly cron job. The backups end up in `<backupdir>/<configname>/nightly`, with hardlinks to `<backupdir>/<configname>/weekly` and `<backupdir>/<configname>/monthly`. You can choose the day to take the weekly and monthly backups, and how many nights, weeks and months back to go, by changing the settings in `backup.ini`, or by overriding these settings in `<configname>.ini`. E.g. if Tuesday is the day you have set for weekly backups, then the nightly tar backup taken on Tuesday will be hardlinked into the weekly directory.

