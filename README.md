# Shipping performant JS code to production

Oh boy, this is a big one. Lots of stuff to cover, across a wide and wildly changing landscape. It's also a topic that covers everyone's favourite: The JS Framewok of the Month™.  I'll try to stick to the "Tools, not rules" mantra and keep the JS buzzwords to a minimum.

But before we dive into specifics, let's dive a bit deeper and get a broader understanding of the issue by answering the following: What is considered as performant JavaScript and how does it fit into the broader web performance metrics?

## Setting the target

First of all, let's get the following out of the way. If you're testing exclusively on your desktop device (you are testing, right?), you're excluding [more than 50%](http://gs.statcounter.com/platform-market-share/desktop-mobile-tablet) of your  users.

![enter image description here](https://i.imgur.com/6XtNnds.png)

This trend will only continue to grow, as the emerging market's preferred gateway to the web is a sub-100$ Android device. The era of the desktop as the main device to access the Internet is over, and the next billion internet users will visit your sites primarily through a mobile device.

Testing in Chrome dev tools' device mode isn't a valid substitute to testing on a real  device. Using CPU and network throttling helps, but it's a fundamentally different beast. Test on real devices.

Even if you are testing on real mobile devices, you're probably testing it on your brand spanking new 600$ flagship phone. The thing is, that's not the device that your users have. The median device is something along the lines of a Moto G1. A device with under 1GB of ram, and a very weak CPU and GPU. To quote Bruce Lawson: "World-

Let's see how it stacks up when [parsing an average JS bundle](https://docs.google.com/spreadsheets/d/1wHcNNQea28LhwQ_amFamT33d5woVrJfJy53Z1k6V090/edit?usp=sharing).

![enter image description here](https://rawgit.com/ivancuric/js-perf/master/Screen%20Shot%202017-11-14%20at%2015.58.55.png)

Ouch. While this image covers the parse and build time of the JS (more on that later) and not general performance, it's strongly correallated.

So, your target for web performance is a device that's **~25x slower** than your MacBook or iPhone. Let that sink in for a bit. But it gets worse.

## What is performant JS code?

Now that we know what our target platform is, we can answer the next question: what _is_ performant JS code? 

While there's no absolute classification of what defines performant code, we do have a user-centric performance model we can use as a reference: [The RAIL model](https://developers.google.com/web/fundamentals/performance/rail).

![enter image description here](https://raw.githubusercontent.com/ivancuric/js-perf/master/rail.png)



## Context is everything 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMzMxMTU1MzFdfQ==
-->