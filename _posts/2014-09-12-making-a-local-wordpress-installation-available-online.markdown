---
layout: post
title:  "Making A Local WordPress Installation Available Online"
date:   2014-09-12 00:00:00 +1100
tags: networking, wordpress
permalink: /making-a-local-wordpress-installation-available-online/
---

It’s generally considered a best practice to develop a WordPress site (or any site, for that matter) in a “development” environment i.e. own your own computer, and transfer the completed site to the “production” environment i.e. the site’s permanent home.

But what if you want to show your in-development site to your client, co-workers etc. who may not be on-site to look at your computer? You can expose your local installation to the internet, using your own computer as a server.

Assuming you’ve already installed WordPress on your local machine with WAMP, MAMP etc, the steps for putting it online are as follows:

1. Identify your local and global IP
2. Unblock port 80
3. Setup your router to forward internet traffic to your site
4. Setup WordPress to work correctly whether viewed either locally or remotely
 

**1. Identify your local and global IP**

There are two separate IP addresses that will be relevant to this setup: the IP address assigned to your router by your ISP (i.e. your global IP) which is the IP used to send/receive information on the internet, and the IP assigned to your computer by your router (your local IP) which is the IP used to send/receive information on your local network.

To identify your global IP, you can just Google “what is my IP” and Google will tell you. Easy.

Note: often your global IP will be dynamic i.e. it changes every time your router connects to the internet via your ISP. This is a problem because other people on the internet won’t know where to find you next time you disconnect and reconnect, as you’ll have a new global IP. Consider contacting your ISP and asking for a static IP which does exactly what you’d expect – remain constant. My ISP offers the service for $10/month. If you proceed with a dynamic IP, you’ll have to change your settings every time you disconnect/reconnect and inform anyone who wants to visit your site, which will be a pain.

The method for identifying your local IP will depend on your operating system, so use Google to find the appropriate method of identifying it.

 

**2. Unblock port 80**

Port 80 is the internet port that web traffic uses. Typically your router (and possibly your ISP) will block incoming traffic to this port because most people don’t need to allow incoming traffic from the internet to their local network, and it’s safer just to keep it blocked.

To see if your port 80 is blocked, use this tool:

[http://www.yougetsignal.com/tools/open-ports/](http://www.yougetsignal.com/tools/open-ports/)

Enter your global IP and 80 as the port. If it’s blocked, go to your router’s admin page and find the security or firewall settings. I don’t recommend turning off the firewall entirely, but usually you can set up a rule to allow traffic just on port 80. Consult your router’s manual for instructions on how to do that.

You may also need to contact your ISP, as they’ll often block port 80 unless you request to have it unblocked.

 

**3. Setup your router to forward internet traffic to your site**

Now you’ve unblocked port 80, and you give someone your global IP and they put it into their internet browser. Still nothing happens. How do you make it so that an internet browser making a request to your global IP will return your locally set up WordPress site?

Firstly, you need to set up port forwarding. Remember it’s your router that has the global IP, not your computer. So right now when people enter your global IP their being directed to the router, which obviously doesn’t have a web page etc to respond with.

You just need to tell your router to forward any traffic on the web from port 80 to your computer. The method for doing this is largely dependent on the model of your router, so you’ll need to ask Google again or consult your router’s manual. Here’s the rough concept so you know what to look for:

1. Enter your router’s admin page on the local network
2. Find the section on Port Forwarding
3. Forward traffic on port 80 of the router to port 80 on your computer (this is where your local IP will be needed). The setting will most likely look like this:Source IP: any. Source port: 80. Destination IP: <your local IP>. Destination port: 80. Protocol: TCP.

Now, when someone enters your IP in their browser, the request will travel to your router, pass the firewall, and your router will now forward that your computer.

 

**4. Setup WordPress to work correctly whether viewed locally or remotely**

If you’ve installed WordPress on your local machine, you’re most likely able to view it in your browser at localhost or localhost/mysite etc. But a remote viewer will not be viewing at localhost, they’ll be viewing via your global IP

The problem is that by default, WordPress usees absolute URLs. You’ll probably know that many files loaded by WordPress will look like this on your local installation:

    <img src="<a href="http://anthonygore.com/path/to/media.jpg">http://anthonygore.compath/to/media.jpg</a>"/>

This is fine when viewed on your local machine, but when someone views the same page online, that image won’t show because “localhost” on their machine is not the same as locahost on your machine.

So couldn’t couldn’t you just change your URL’s to your global IP like the following?

    <img src="<a href="http://anthonygore.com/path/to/media.jpg">http://59.167.3.220/path/to/media.jpg</a>"/>
    
No, the problem with that is your browser will route the request for that image out onto the internet and back to your machine, pointlessly taking additional time to load the image.

The solution is to use relative URLs instead of absolute URLs like those above e.g.:

    <img src="/<a href="http://anthonygore.compath/to/media.jpg">path/to/media.jpg</a>"/>
    
You need to make some changes to your site for this to work, though. Note this should only be done on a development server, don’t do this in production as it can have other negative consequences.

Firstly, add the following to your index.php file:

    $your_ip = "59.167.3.220"; // change this to your global IP
    if ($_SERVER['SERVER_NAME'] == $your_ip) {
        define('WP_HOME', $your_ip);
        define('WP_SITEURL', $your_ip);
    }
    
This means that if someone is requesting your site via your IP, WordPress will use it as the site URL. If not, it will stick with the default (i.e. localhost).

You’ll also need to install plugins to ensure WordPress knows to use relative URLs. Try these:

[https://wordpress.org/plugins/relative-url/](https://wordpress.org/plugins/relative-url/)

[https://wordpress.org/plugins/relative-image-urls/](https://wordpress.org/plugins/relative-image-urls/)

If you’re applying this to an existing site, you might have some hard-coded links you’ve added which may have to be changed manually, so ideally you want to set the above up before you’ve started working on the site.

Now your clients, co-workers etc should be able to view your WordPress site, just as you do, through `http://your-global-ip`.

If you want to go a step further, you can even assign a domain name to your global IP. For example, `http://www.yourname.com/test`. That’s a topic for another post.
