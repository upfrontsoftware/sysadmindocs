==============
Backup process
==============

Contents:

.. toctree::
   :maxdepth: 2

-------------
Infrastucture
-------------

Upfront backuptools
~~~~~~~~~~~~~~~~~~~

    repo at: https://svn.upfronthosting.co.za/svn/backuptools

    tarbackup.py

    - periodic tar backups of configured files/directories
    - each day incremental backup
    - every 7 days (default settings); full backup

    ftpsync

    - move files to remote server
    - uses ftp
    - comparison based on file name + file size

Repozo
~~~~~~

    backup data.fs of zope/plones

    repozo does full backup the first time, and then incrementals until next pack

Cron
~~~~

    Triggers the process on a timed basis

Process
~~~~~~~

    Cron config in /etc/cron.d/backup

    Cron fires the script /usr/local/sbin/backup-all.sh

    Script uses repozo and/or tarbackup to make backups

    Then script moves files offsite with ftpsync

    Finally it contacts nagios and tells it that the backup succeeded

Servers
~~~~~~~

    Flotron

    Okacom

    Siyavula

        MySQL database

        Wordpress plugin 
