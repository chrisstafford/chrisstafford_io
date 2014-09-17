---
layout: post.html
title: Starting out with Ghost
tags: [ghost, nodejs]
---

** It should be mentioned, that this site no longer runs on ghost. I have switchted over to [mynt](http://mynt.uhnomoli.com/) **

After waiting patiently it looks like Ghost has finally released their code to the public, so I figured I'd give it a shot. After running through the documentation over at [http://docs.ghost.org/](http://docs.ghost.org/) setup was quick and painless. Just clone the github repository ([https://github.com/TryGhost/Ghost](https://github.com/TryGhost/Ghost)), install the dependencies, run grunt init and start the app.

I didn't run into any snags until I decided I wanted to run it at "production" level as my main blog. I found the configuration pretty straight forward, just enter your values in the production hive. Setting up nginx to accept http request was just like doing it with any other nodejs app, just create an upstream proxy and point it to the port you are running Ghost on.

It wasn't until I tried running Ghost while passing the NODE_ENV=production environment variable. The app started fine, but when I went to navigate to the backend the app would warn me that I didn't have javascript enabled, after glancing at Chrome's dev tools it was complaining that it couldn't find `core/build/scripts/ghost.min.js` I looked in the directory and found no minified version of the file. I can only assume that when you run Ghost at production it automatically looks for the minified version. To solve this I installed the minifier node package ([https://npmjs.org/package/minifier](https://npmjs.org/package/minifier)) and pointed it at `core/build/scripts/ghost.js` and it created the minified file and all was well. So far all I've done is install and write this post, but I like it. It's simple and elegant, and functions well. I'd like to find a way to add disqus for comment functionality, but I'm sure something is in the works.

**Edit:** I found a write up here [http://blog.christophvoigt.com/enable-comments-on-ghost-with-disqus/](http://blog.christophvoigt.com/enable-comments-on-ghost-with-disqus/) on how to add disqus to the blog and will add it soon.

Also it looks like you can run `grunt prod` to generate the minified file. Looks like I should have read all of the documentation ;).
