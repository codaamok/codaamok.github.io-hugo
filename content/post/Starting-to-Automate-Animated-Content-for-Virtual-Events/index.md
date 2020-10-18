---
title: "Starting to Automate Animated Content for Virtual Events"
date: 2020-07-12T00:00:00+01:00
draft: false
image: images/cover.jpg
categories:
    - Presentations
---

I recently took ownership of becoming the organiser for my local PowerShell user group - [PowerShell Southampton](https://pssouth.co.uk). With everything going on in the world re the pandemic, I realised I won't be arranging in-person meetups any time soon.

I got stuck in and learnt ways I could produce a virtual event: looked at the tech available and weighed up the features, pros/cons etc with how I wanted it to look and what I wanted the user experience to be for all involved (organiser, speaker and audience).

The research was short-lived. I quickly landed on using OBS Studio and pushing the stream to YouTube. OBS Studio seems like the de facto choice for screen recording and streaming. After some hours tinkering, reading and doing my owns streams to [Twitch](https://twitch.tv/codaamok) (archived to my [YouTube channel](https://www.youtube.com/channel/UCjv4o4ToaauMNKP4wfIhP0A)), I was more than happy with what it offered.

Here is some material I used to help me learn streaming: a bunch of posts by [Chrissy LeMaire](https://blog.netnerds.net/category/livestreaming/) and a small list of resources stored on the [PowerShellLive repoitory](https://github.com/powershelllive/LiveStreamQuickStart/blob/master/README.md).

The first event is this Wednesday and I got seriously carried away this weekend with styling some "starting soon" or "be right back" animations. Below is what I've ended up with and I'll quickly walkthrough how to use it if you wanted to do something similar.

{{< youtube VAtMaMHLFW8 >}}

As you can see in both scenes there is only 1 single browse source. This immediately makes my OBS setup super simple and easily to recreate if I ever need to.

There's just two resources needed to get rolling with this, and [they're both in a Gist](https://gist.github.com/codaamok/b0c1f6b203de03c18ed91268c31520b4):

- `index.php`
- `New-PSSouthamptonVirtualSession.ps1`

The browser source in OBS points to `http://127.0.0.1/pssouthampton/index.php?message=We'll be right back, sorry for the interruption!`. Pass a difference value to the message parameter in the query string of the URL to change the scrolling text.

Unfortunately, this does depend on XAMPP (or similar) for at least a local web service and PHP. I was fixed on the idea of reading from a .json file within the HTML which contained properties such as session title, speaker name, Twitter handle, Twitter image path. I tried many ways of doing this with JavaScript but no dice, in the end I gave up and accepted the dependency.

`index.php` hugely depends on the browser's size (or OBS's canvas size) being 1920x1080. You'll also notice that if you opened `index.php` in a browser, the positioning of some items might look different. I assume that's either because the display scaling of my 4k display set to 175%, or the browser engine in OBS is different in some way.

Accompanying that is a PowerShell script `New-PSSouthamptonVirtualSession.ps1`. This is the only thing I intend to use before each new event. I pass parameters for session title, speaker's name and Twitter handle, and it does the rest:

- Updates/creates properties.json with said properties, which `index.php` reads from
- Downloads the speaker's Twitter profile and specifies its filename
- Starts XAMPP's Apache if not already running

This workflow makes life very easy moving forward: no need to fiddle with OBS sources and positioning them, or manipulating images in some editor. At most I'll likely only need to fiddle with the session title's text size and positioning in the CSS of `index.php` if it gets particularly lengthy.

Here's a quick demo of using the `New-PSSouthamptonVirtualSession.ps1` and how easily it changes `index.php`:

{{< youtube 1OhdfvRGNyo >}}
