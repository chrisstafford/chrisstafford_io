---
layout: post.html
title: Using an internal proxy for wordpress development
tags: [proxy-server, squid, wordpress]
---

One of the products I have to support is custom wordpress development. Generally what we do is design the site and create mockups in house, then we send those out to contractors to do the actual wordpress development. This generally works well until we get the site back and need to add content to round out the site.

We used to have a development server that we would build the sites on and would push changes down to our load balanced production web servers. This works well most of the time, we only run into trouble when the them we end up using tries puts the full path of images or other static content into the database. We are then left with a whole bunch of content pointing back to our dev servers. I would then have to go in and change the values by hand in the database. Which as you can imagine is far less then ideal.

My first fix was to actually set wordpress to use the actual domain name and place host file enteries on the computers of the people doing the content entry in house. This became an issue because a lot of the time they would just copy and paste content from the live site, and they really didn't have any idea how to activate and deactivate entries in their hosts files. Not to mention this was just a poor workflow overall.

My next approach will now be to set up a proxy server in house that will have custom hosts file entries I can create (all in one place as opposed to every machine working on the project) and take on of the browsers they are using and point it to the proxy server. This will allow them to have both the current live site open in one browser and connect to the dev site in the other. I will using squid to accomplish this, and will post a tutorial of how I accomplished it once I have as well as a follow up on how well it worked out for us.

How would you approach this situation?
