---
layout: post
title:  "Which image did I just load into that UIImage?" 
date:   2015-09-15 15:33:41
description: How can I debug a UIImage from XCode? Debugging UIImages to see the loaded image from XCode.
#categories: ios programming development UI UX
tags: ios programming development UI UX
thumbnail: /images/debug-image.png
---

It's been a couple of weeks since my last post. In this brief post I am going to show you how to check out what image you've loaded into your UIImage. I found this helpful when loading image assets from a REST webservice. So first thing's first I set up a breakpoint after my UIImage is initialized.

![XCode Breakpoint at UIImage initialization]({{ site.baseurl }}/images/set-uiimage-in-xcode.png){: .img-responsive .center-block }

Then by clicking the *eyeball* icon we get a nice preview of the image. Pretty neat. 

![Preview UIImage in XCode]({{ site.baseurl }}/images/preview-uiimage-in-xcode.png){: .img-responsive .center-block }

I know this probably won't be incredibly useful to lots of folks but hopefully it comes in handy for someone that's struggling to make sure they have the right image at the right time.