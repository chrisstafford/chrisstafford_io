---
layout: post.html
title: Internal Proxy for Wordpress Development Part 2 (setup)
tags: [proxy-server, squid]
---

As I mentioned in part 1 ([http://ghost.chrisstafford.io/using-an-internal-proxy-for-wordpress-development/](http://ghost.chrisstafford.io/using-an-internal-proxy-for-wordpress-development/)) I've decided to setup an internal proxy server to aid in building new wordpress sites.

First step was to spin up a new virtual server and install ubuntu on it. From there I went through and followed the instructions in the official ubuntu documentation [https://help.ubuntu.com/lts/serverguide/squid.html](https://help.ubuntu.com/lts/serverguide/squid.html). At first glance the squid documentaion is pretty overwhelming, so I would recommend backing up the included squid config and creating a blank squid.conf. From there just add the config values you need. Mine is pretty simple as it is only internal and is only needed to control dns for a very small number of sites. It looks like this.

    http_port 8888 #something other than the default port 3128
    visible_hostname devproxy
    acl local_network src 192.168.1.0/24
    http_access allow local_network
From here I needed a way for the people entering the content to easily switch the proxy on and off. Luckily there are tons of browser extensions out there to handle this for you. Because everyone in the office uses chrome I went with Proxy Switchy Sharp [https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm?hl=en](https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm?hl=en). I installed it into all of there browsers and built a profile that points to the proxy server and showed them how to use it. Now whenever we have a new site I just add the entry into the hosts file of the proxy server and restart squid and they can use the extension to access the site an make whatever edits they need before the site goes live.

Today was day 0 so if they decide they like I'm hoping to add some authenication to the server and write up some documention so our clients can look at their sites while in development.
