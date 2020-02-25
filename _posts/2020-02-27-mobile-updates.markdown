---
layout: post
title: "Mobile Updates to the Blog!"
categories: Posts
author: Austin Webre
---
So recently, I finally got around to fixing the last couple of podcast platforms we had yet to join. It was at this point that I realized this site was missing something huge: links to our episodes on every platform! So I took some time to add links with brand images to the top man, at which point I proudly messages Clinton to get what I assumed to be resounding praise. Instead I got, "hmmm... they look too small." Not what I was looking for. So I made them as big as I could while maintaining some sense of hierarchy, but apparently they were still too small. It was at this point that I realized he was on his phone. 

This is where the rabbit hole begins. I start looking over the site on mobile and realize a couple of things. For one, Clinton was right, on desktop the icons looked perfect but on a phone, they were hard to make out and even harder to click with any precision. The second thing I noticed was that our embedded player had completely broken the page on mobile. Now most people probably would've stayed focused on the problem at hand, but to me the web player was a bigger issue. It's a huge reason for the site's existence and it was breaking every single page it was on, which is almost every single page on the site. So I started down the path of replacing our live player.

Now most podcast distributors have their own embedded player. Anchor (our distributor) does, but it's the one that was previously discussed. It's a really sleek looking player, but it's just not mobile friendly, period. So I began my research and there are so many open source podcast web players... for Wordpress. This of course, is a GitHub Pages site, which uses Jekyll (an open source, markdown based blogging framework with liquid templating). After digging through all the Wordpress and Drupal plugins, I came across PodLove's Webplayer. It's everything we needed - open source, mobile responsive, highly configurable, and served from a CDN. 

Now, the documentation was lacking some (as so much open source documentation is), but after dabbling with the settings for a little while, I had all the information we wanted in a test player on one of the episode pages. From there, it was a matter of pulling out the epsisode specific settings (episode titles, duration, etc) into Jekyll's per page yaml settings, called "front matter" for reasons unknown to me. 
