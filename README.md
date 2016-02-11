CMS Backup script
=================
This is a script, `backup`, which handles backups of a content management system. I.e. some files plus one or more databases. The files can be specified quite flexibly as paths to include or exclude relative to a top level directory given in the global config file. The databases can be (currently) either mysql or postgresql. There is also another script, `pullbackups`, which will pull down backups generated this way from a remote server. The bulk of the work done by both scripts is handled by `rsync`.

The backup script
-----------------
Each backup consists of a directory containing all of the backed up files and databases. The directory name gives the name of the backup configuration being run, plus a date stamp. Identical files between backups are hard links to the same inode; this means that each successive backup only uses the extra disk space taken up by the changed or new files. This is like incremental backups but better because every copy is a master copy. (You need to run the script on a filesystem that supports hard links).

It can be invoked in either manual or automatic mode. Manual mode makes a single backup with a date stamp; automatic mode can be called from a cron job to generate a series of nightly, weekly, and monthly backups. For example, it can be set so that you can go back in steps of a day for seven days, a week for four weeks, and a month for six months.

Pulling remote backups
----------------------
A similar scheme is used by the pullbackups script. You can pull backups generated by the first script from several remote server accounts and archive them in a series of directories. Each archive contains date-stamped sub directories with nightly, weekly, and monthly mirrors of the backups on the given server account. Again, hard links are used to reduce disk space. The way rsync is invoked should enable it to detect and pull down only (compressed) changes to the databases on the remote end, rather than the whole database dump every night. This saves on network traffic.

Installation
------------
To install, first clone the repository with `git clone --recursive https://github.com/highfellow/cms-backup-script.git` to a suitable location (e.g. `/usr/local/src`)

To set up the `backup` script, symlink `backup` to `/usr/local/bin/backup` and copy `backup.ini` to `/usr/local/etc/backup.ini`. Make a directory `backup.d` in `/usr/local/etc`; this contains any number of `<configname>.ini` files, which define a particular set of files and databases to back up. `example.ini` in this directory can be used as a template. Edit the `.ini` files as necessary. For more information, see the comments in `backup.ini` and `example.ini`. 

To set up `pullbackups`, you need to do much the same thing. Symlink `pullbackups` to `/usr/local/bin/pullbackups` and copy `pullbackups.ini` to `/usr/local/etc/pullbackups.ini`. Make a directory `pullbackups.d` in `/usr/local/etc` containing `<remotename>.ini` files, using `pb-example.ini` as a template.

You also need to set up your remote account to allow password-less logins from the machine you are installing `pullbackups` on. The normal way of doing this is to copy your ssh public key from the local machine to `~/.ssh/authorized_keys` in the home directory of the remote account.

If you will be running the script(s) in automatic mode, you'll also need to copy the relevant `.logrotate` files to `/usr/local/etc` and edit as necessary.

Invocation - backup
-------------------
You invoke the script as `backup [--auto] [--summarise] [<configname>] ...`. (Note more than one config is possible.). `-a` and `-s` are synonyms for `--auto` and `--summarise`. E.g. `backup -a myblog clientsite` to take a nightly backup of your personal blog and a site you have done for a client. In auto mode, the output is logged to files, and a weekly summary log is also generated which can be mailed out by calling `backup --summarise`.

If there is no `-a` you are in manual mode, and the backup(s) will end up in `<backupdir>/<configname>/manual` where `<backupdir>` is the backup directory you specified in the global config file.

With the `-a`, you are in automatic mode, which is intended to be used from a nightly cron job. The backups end up in `<backupdir>/<configname>/nightly`, with hardlinks to `<backupdir>/<configname>/weekly` and `<backupdir>/<configname>/monthly` where appropriate. You can choose the day to take the weekly and monthly backups, and how many nights, weeks and months back to go, by changing the settings in `backup.ini`, or by overriding these settings in `<configname>.ini`. E.g. if Tuesday is the day you have set for weekly backups, then the nightly tar backup taken on Tuesday will be hardlinked into the weekly directory.

Invocation - pullbackups
------------------------
You run this script using `pullbackups [--auto] [--summarise] [--stop] [<remotename>] ...`, where `<remotename>` is a name given to a remote account, and `-a`, `-s`, and `-p` are synonyms for `--auto`, `--summarise`, and `--stop`. E.g. `pullbackups -a me@blogserver.com`. There should be a file `/usr/local/etc/pullbackups.d/<remotename>.ini`. Again, you can customise how many nightly, weekly, and monthly backups are taken.

The `--summarise` option will mail the weekly summary log to one or more addresses given in `pullbackups.ini`.

The `--stop` option allows you to safely stop a running pullbackups process, so as to prevent it from running into the next day and hogging daytime bandwidth. You just need to call `pullbackups --stop` from a cron job.

Automatic invocation
--------------------
Both programs can be called from a cron job to automate taking nightly backups and pulling them down to an off site machine. Below are example crontab entries with comments.

Crontab for `backup`:
```
# backup a personal blog and a client website to the local archive.
30 0 * * * /usr/local/bin/backup -a myblog clientsite
# every tuesday at 10 am, mail out a weekly summary of the results of the backup.
0 10 * * 2 /usr/local/bin/backup --summarise
# rotate the log files. Note that if logrotate is run as a non-root user, it needs to be given a path to a file
# it can store its state in.
0 0 * * * /usr/local/bin/logrotate -s /var/local/lib/logrotate/status /usr/local/etc/backup.logrotate
```

Crontab for `pullbackups`:
```
# pull down backups from the machine running the backup script.
30 0 * * * /usr/local/bin/pullbackups -a me@remote.org
# every morning, stop the pullbackups process if necessary
0 8 * * * /usr/local/bin/pullbackups --stop
# every tuesday at 10 am, mail out a weekly summary of the results of the backup pull.
0 10 * * 2 /usr/local/bin/pullbackups --summarise
# rotate the log files. Note that if logrotate is run as a non-root user, it needs to be given a path to a file
# it can store its state in.
0 0 * * * /usr/local/bin/logrotate -s /var/local/lib/logrotate/status /usr/local/etc/pullbackups.logrotate
```
