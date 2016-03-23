---
layout: post
title:  "WordPress + Git: Version Control Your MySQL Database (With Instructions For Vagrant)"
date:   2014-09-22 00:00:00 +1100
tags: bash, git, mysql, vagrant, wordpress
permalink: /wordpress-database-git-simple-way-to-version-control-your-mysql-database-with-instructions-for-vagrant
---

One of the problems when putting your WordPress project under version control is that the files alone are somewhat meaningless without the database. You can’t really rollback to a previous commit and expect it to work if the database has changed.

One possible solution is to put your database under version control as well. An easy way to do it is to put a bash script in your root folder like this:

    #!/bin/bash
    USER="wordpress"
    PASS="wordpress"
    DB="wordpress"
    mysqldump -u $USER -p$PASS $DB --add-drop-table > dbbackup.sql
    
Save the file as `dbbackup.sh` and you can simply run `./dbbackup.sh` before git add when you’re about to stage and commit your files. This will mean your repository is a snapshot of the whole WordPress project, not just the files.

You can also create a script to restore the database. Save this as `dbrestore.sh` and run it after a `git pull`:

    #!/bin/bash
    USER="wordpress"
    PASS="wordpress"
    DB="wordpress"
    mysqlimport -u $USER -p$PASS $DB dbbackup.sql
    
**Vagrant**

If you use Vagrant, which I recommend, you’ll need to login to the guest box to do the database dump, otherwise the script will think you’re trying to run `mysqldump` from the local MySQL. This will do the trick:

    #!/bin/bash
    vagrant ssh -c '
    USER="wordpress"
    PASS="wordpress"
    DB="wordpress"
    mysqldump -u $USER -p$PASS $DB --add-drop-table > /var/www/anthonygore.com/public_html/dbbackup.sql
    '
    
`vagrant ssh -c ''` will run the command(s) inside the quotes from your vagrant box’s terminal.

**Security**

I also put the following in my .htaccess file, to ensure, if the database backup gets deployed to a production server, that no one can open these files via the browser. Correct file permissions will do the job, but those tend to go awry when git’ing.

    <FilesMatch "(dbbackup\.sql|dbbackup\.sh|dbrestore\.sh)$">
        Order Allow,Deny
        Deny from all
    </FilesMatch>
