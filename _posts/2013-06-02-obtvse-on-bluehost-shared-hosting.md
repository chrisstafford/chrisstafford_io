---
layout: post.html
title: Obtvse on Bluehost shared hosting
tags: [bluehost, obtvse, ruby, rails]
---

After a few days of hacking around I was finally able to get obtvse up and running and I've decided to create a how-to for those who are also having issues.

First things first, let's clone obtvse onto our server.
```bash
    git clone http://github.com/NateW/obtvse.git
```
You'll notice that we are cloning over http as opposed to git.  This is because Bluehost only opens port 80 (http) and 443 (https).

After the clone has completed we need to change directory to ~/obtvse
```bash
    cd obtvse
```
Before we can run bundle install we need to edit our Gemfile.
```bash
    vim Gemfile
```
Because we are pulling stringex from github we need to change the git protocol to http by replacing
```bash
    gem 'stringex', '~> 1', git: 'git://github.com/rsl/stringex.git'
```
with
```bash
    gem 'stringex', '~> 1', git: 'http://github.com/rsl/stringex.git'
```
We can now run
```bash
    bundle install
```
Before we run rake db:migrate we first need to install node.js  I turned to this how-to accomplish this.

**Note:** I found that I needed to install node.js into the apps directory (~/obtvse).  To accomplish this change your configure prefix parameters.
```bash
    ./configure --prefix=/home/{username}/node
```
to
```bash
    ./configure --prefix=/home/{username}/obtvse/node
```
[http://rcrisman.net/article/10/installing-nodejs-on-hostmonster-bluehost-accounts](http://rcrisman.net/article/10/installing-nodejs-on-hostmonster-bluehost-accounts)

From here we can now run
```bash
    rake db:migrate
```
You are now going to want to edit the config file as outlined in README.md

Bluehost uses Passenger to run ruby on rails apps, to get Passenger to recognize and serve your app you need to edit the .htaccess file in the ~/public_html directory.
```bash
    vim ~/public_html/.htaccess
```
Add this to the top of the file.
```bash
        Options -MultiViews
        PassengerAppRoot /home1/{username}/obtvse
        PassengerResolveSymlinksInDocumentRoot off
        #Set this to whatever environment you'll be running in
        RailsEnv development
        RackBaseURI /
        SetEnv GEM_HOME /home1/{username}/ruby/gems
```
Save and hit your domain from your browser.
