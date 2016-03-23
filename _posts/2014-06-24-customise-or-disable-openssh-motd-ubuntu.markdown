---
layout: post
title:  "Customise Or Disable OpenSSH MOTD, Ubuntu"
date:   2014-06-24 00:00:00 +1100
tags: linux
permalink: /customise-or-disable-openssh-motd-ubuntu/
-----------------------------------------------------

When you login to your Ubuntu server with SSH you may see a message like this:

    Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.11.0-23-generic x86_64)
     
    * Documentation:  https://help.ubuntu.com/
     
    Last login: Tue Jun 24 21:00:01 2014 from 123.123.123.123

It's called the Message Of The Day (MOTD). It's created by running, in numerical order, the scripts in /etc/update-motd.d. On my system I have:

    00-header
    10-help-text
    90-updates-available
    91-release-upgrade
    98-fsck-at-reboot
    98-reboot-required
    99-footer

You can of course edit those scripts or add more. How can you disable the MOTD? Firstly ensure you have this in /etc/ssh/sshd_config:

    PrintMotd no

If you’re using PAM, you may still see the message, despite turning it off in sshd_config. In which case edit /etc/pam.d/sshd:

    #Print the message of the day upon successful login.
    #session    optional    pam_motd.so

Be sure to comment out the second line.

In my case, I don’t want connecting users to see the dynamically created MOTD, I just want a plain, old banner with a legal notice, something like this:

    WARNING<strong> :</strong> Unauthorized access to this system is forbidden and will be
    prosecuted by law. By accessing this system, you agree that your actions
    may be monitored if unauthorized usage is suspected.

For the more creative, why not use a cool [ASCII text banner](http://www.kammerl.de/ascii/AsciiSignature.php)?

When you’ve made your banner, save it to /etc/ssh/sshd-banner (you can specify a different directory if you like), and edit /etc/ssh/sshd_config again and add this line:

    Banner /etc/ssh/sshd-banner

Be sure to then restart SSHD:

    $ sudo service ssh restart
