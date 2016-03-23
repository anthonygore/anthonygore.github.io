---
layout: post
title:  "Automated Backup Of AWS Route 53 Record Sets"
date:   2015-02-16 00:00:00 +1100
tags: aws, bash
permalink: /automated-backup-aws-route-53-records
---

If you’re using AWS Route 53 to manage DNS records, it’s a good idea to backup in case of accidental deletion and other such misfortunes.

Of course you’ll want it automated so here’s a way to do with cron on a linux system:

1.  Install the [AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/installing.html). This tool allows you to administer your various AWS services via the command line.
2.  Install [cli53](https://github.com/barnybug/cli53). This tool extends the AWS CLI by offering more high-level commands for easy Route 53 administration
3.  Once you have those setup, the following command will export a zone record to a file:

        $ cli53 export example.com --file example.com.bk

4.  You need to specify what domain you want the zone record for, there’s no “all” option. So, you could go ahead an run the command repeatedly for all your domains, but who wants to do that? To do it programmatically, this following command will get the list of domains, iterate through them, and export each one, piping the result to a separate file:

        cli53 list | grep 'Name:*' | cut -f6- -d' ' | while read line; do cli53 export ${line} >> ~/backup/${line}bk; done

5.  To have this happen automatically, you can simply create a bash script and have cron run it once per day or whatever you like:

        #!/bin/bash
        cli53 list | grep 'Name:*' | cut -f6- -d' ' | 
        while read line; 
        do
            cli53 export ${line} >> ~/backup/${line}bk; 
        done

    Note: the `cli53` command won’t work in a bash script unless you provide the full path or add to the `$PATH` variable e.g `/usr/local/bin/cli53`

6.  Save it, let’s say to _/path/to/script.sh_, make that file executeable, and add it to cron:

        $ crontab -e

    Add this to the bottom of the file to run the script once per day:

        00 00 * * * sh /path/to/script.sh

Note that your backups will be overwritten each time the script runs, so you might add a date to the file name to create daily snapshots.Or better still: why not upload your backup files to a versioned S3 bucket for safe storage? That also be done with AWS CLI, here’s a modified version of the bash script to do just that:

        #!/bin/bash
        cli53 list | grep 'Name:*' | cut -f6- -d' ' | 
        while read line; 
        do
            cli53 export ${line} > ~/backup/${line}bk;
            aws s3 cp ~/backup/${line}bk s3://mybucket;
            rm ~/backup/${line}bk;
        done
