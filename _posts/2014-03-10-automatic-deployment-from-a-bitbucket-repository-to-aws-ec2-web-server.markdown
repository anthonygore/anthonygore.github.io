---
layout: post
title:  "Automatic Deployment From A Bitbucket Repository To AWS EC2 Web Server"
date:   2014-03-10 00:00:00 +1100
tags: aws, git, linux, php
permalink: /automatic-deployment-from-a-bitbucket-repository-to-aws-ec2-web-server/
---


I had quite a bit of trouble setting up automatic deployment of a git repo to a Linux instance on EC2.

I’m using the PHP script outlined in the blog post below, which is triggered by Bitbucket’s POST web hook:

[http://brandonsummers.name/blog/2012/02/10/using-bitbucket-for-automated-deployments/](http://brandonsummers.name/blog/2012/02/10/using-bitbucket-for-automated-deployments/)

But my log contained the following error when a git pull was attempted:

    Could not create directory '/var/www/.ssh'. Host key verification failed. fatal: Could not read from remote repository.  Please make sure you have the correct access rights and the repository exists.
  
The problem is that the PHP script runs as user apache, which has no SSH key setup.

The solution:

1. From the Linux command line, give user apache shell access. Without this, you can’t generate an SSH key to get access rights to the repo. This can be done by editing /etc/passwd

        sudo nano /etc/passwd
   
    Then change line:
    
        apache:x:48:48:Apache:/var/www:/sbin/nologin
    
    To:
    
        apache:x:48:48:Apache:/var/www:/bin/bash
        
2. Create directory /var/www/.ssh

        sudo mkdir -p /var/www/.ssh/

3. Change the owner of directory to user apache

        sudo chown -R apache /var/www/.ssh/

4. Switch to user apache

        su - apache

5. Generate an SSH key

        ssh-keygen

    Leave the password blank.

6. Copy the SSH key and put it in Bitbucket

        cat /var/www/.ssh/id_rsa.pub

    And add to Bitbucket’s SSH keys.

7. Change back to the root user and remove shell access to user apache.
   
        sudo nano /etc/passwd

    Change line:
    
          apache:x:48:48:Apache:/var/www:/bin/bash
              
    Back to:
    
        apache:x:48:48:Apache:/var/www:/sbin/nologin

And the deploy script now works.

Further reading:

[http://jondavidjohn.com/git-pull-from-a-php-script-not-so-simple/](http://jondavidjohn.com/git-pull-from-a-php-script-not-so-simple/)

[http://stackoverflow.com/questions/7306990/generating-ssh-keys-for-apache-user](http://stackoverflow.com/questions/7306990/generating-ssh-keys-for-apache-user)

[http://stackoverflow.com/questions/9370975/running-git-pull-from-a-php-script](http://stackoverflow.com/questions/9370975/running-git-pull-from-a-php-script)

[http://serverfault.com/questions/362012/running-git-pull-from-a-php-script](http://serverfault.com/questions/362012/running-git-pull-from-a-php-script)

[http://stackoverflow.com/questions/5144039/shell-exec-and-git-pull](http://stackoverflow.com/questions/5144039/shell-exec-and-git-pull)

[http://stackoverflow.com/questions/9370975/running-git-pull-from-a-php-script](http://stackoverflow.com/questions/9370975/running-git-pull-from-a-php-script)
