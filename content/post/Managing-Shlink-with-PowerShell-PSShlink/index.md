---
title: "Managing Shlink With PowerShell PSShlink"
date: 2020-11-29T17:56:38Z
draft: true
image: images/cover.jpg
categories:
    - PowerShell
---

I want to share with you a PowerShell module I recently wrote called PSShlink!

It is a module which essentially provides wrapper functions for Shlink's API. Shlink is an open-source self-hosted and PHP-based URL shortener application. 

Shlink helped me tremendously when I moved my domain and blog CMS from cookadam.co.uk to adamcook.io. Some of my posts rank modestly in search results for some keywords and I felt unhappy about letting that go. I also did not want people's bookmarks for posts using my old domain to result in an incomplete redirect to its new URL.

Google also help you secure your position in search results if you tell them your new domain. There were a couple of prerequisites to make this happen and one of them was to ensure all posts (discovered by my sitemap.xml) provided a 301 to a valid URL, usign my new domain. This is where Shlink stepped in.

Any way, I wholeheartedly consumed Shlink which enabled me to seamless migrate my blog and I loved it. Thankfully I didn't have too many posts to carry over from WordPress to Hugo. But there was enough to make me want to use a tool to convert WordPress post XML to Markdown: [lonekorean/wordpress-export-to-markdown](https://github.com/lonekorean/wordpress-export-to-markdown).

There was also enough to make my hands hurt with a lot of careful copying and pasting trying to create all the short links in Shlink for both domains `cookadam.co.uk` and `www.cookadam.co.uk`. I looked to see if there was a Shlink module in PowerShell gallery, and there wasn't. Like many other PowerShell enthusiasts, I jumped on the opportunity and wrote [PSShlink](https://github.com/codaamok/PSShlink).


