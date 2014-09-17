---
layout: post.html
title: Opening up our dev site proxy for external users.
tags: [proxy-server]
---

After successfully deploying a squid proxy server internally I've decided to move forward and make the proxy avaialable externally so people working remotely and our customers can view the sites while they are in progress. My biggest concern with make the proxy public was unauthorized utilizing the proxy so I had to make a few minor changes to my squid.conf to add digest authentication.

First we need to add the following two lines to enable authentication and to add a relam in which to add users to.

    auth_param digest program /usr/lib/squid3/digest_pw_auth -c /etc/squid3/passwords

    auth_param digest realm public

Next we need to add a new group to the access control list and ensure that authentication is required. We can do this in just one line.

    acl public proxy_auth REQUIRED

Finally allow the public group to connect via http

    http_access allow public

Now in the line where we enabled authentication we also told squid to retieve our users from a password file located in `/etc/squid3/passwords`. We can create this file and a user using the `htdigest` tool, which would look something like this.

    sudo htdigest -c /etc/squid3/passwords public user

This command will create `-c` the file specified `/etc/squid3/passwords` and add a user `user` to the realm `public`. Subsequent users can be added by dropping the `-c`.

    sudo htdigest /etc/squid3/passwords public user2

I've sent some rough instructions over to our marketing/support team to write up some customer documentation on connecting to our proxy using Firefox (foxyproxy), Chrome (Proxy SwitchySharp) and IE's built in proxy settings. We'll see if this works out.
