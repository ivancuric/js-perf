# Shipping performant JS code to production

Oh boy, this is a big one. Lots of stuff to cover, across a wide and wildly changing landscape. It's also a topic that covers everyone's favourite: The JS Framewok of the Month™. 

Let's start by answering the following: What is considered as performant JavaScript and how does it fit into the broader web performance metrics?

## Setting the target

First of all, let's get the following out of the way. If you're testing exclusively on your desktop device (you are testing, right?), you're excluding [more than 50%](http://gs.statcounter.com/platform-market-share/desktop-mobile-tablet) of your  users.

![enter image description here](https://i.imgur.com/6XtNnds.png)

This trend will only continue to grow, as the emerging market's preferred gateway to the web is a sub-100$ Android device. The era of the desktop as the main device to access the Internet is over, and the next billion internet users will visit your sites primarily through a mobile device.

Testing in Chrome dev tools' device mode isn't a valid substitute to testing on a real  device. Using CPU and network throttling helps, but it's a fundamentally different beast. Test on real devices.

Even if you are testing on real mobile devices, you're probably testing it on your brand spanking 600$ flagship phone. The thing is, that's not the device that your users have. The median device is something along the lines of a Moto G. Let's see how it stacks up when [parsing an average JS bundle](https://docs.google.com/spreadsheets/d/1wHcNNQea28LhwQ_amFamT33d5woVrJfJy53Z1k6V090/edit?usp=sharing).

![enter image description here](https://rawgit.com/ivancuric/js-perf/master/Screen%20Shot%202017-11-14%20at%2015.58.55.png)

Ouch. While this image covers the parse and build time of the JS (more on that later) and not general performance, it's strongly correallated.

So

## Context is everything 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NDg4NTY4NjBdfQ==
-->