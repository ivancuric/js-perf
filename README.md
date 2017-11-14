# Shipping performant JS code to production

Oh boy, this is a big one. Lots of stuff to cover, across a wide and wildly changing landscape. It's also a topic that covers everyone's favourite: The JS Framewok of the Month™. 

I'll try to stick to the "Tools, not rules" mantra and keep the JS buzzwords to a minimum. Since I won't be able to cover everything related to JS performance in a 2000 word article (duh.), make sure you read the references and do your own research afterwards.

But before we dive into specifics, let's dive a bit deeper and get a broader understanding of the issue by answering the following: What is considered as performant JavaScript and how does it fit into the broader scope web performance metrics?

## Setting the stage

First of all, let's get the following out of the way; If you're testing exclusively on your desktop device, you're excluding [more than 50%](http://gs.statcounter.com/platform-market-share/desktop-mobile-tablet) of your users.

![enter image description here](https://i.imgur.com/6XtNnds.png)

This trend will only continue to grow, as the emerging market's preferred gateway to the web is a sub-100$ Android device. The era of the desktop as the main device to access the Internet is over, and the next billion internet users will visit your sites primarily through a mobile device.

Testing in Chrome dev tools' device mode isn't a valid substitute to testing on a real  device. Using CPU and network throttling helps, but it's a fundamentally different beast. Test on real devices.

Even if you are testing on real mobile devices, you're probably testing it on your brand spanking new 600$ flagship phone. The thing is, that's not the device that your users have. The median device is something along the lines of a Moto G1. A device with under 1GB of ram, and a very weak CPU and GPU.

Let's see how it stacks up when [parsing an average JS bundle](https://docs.google.com/spreadsheets/d/1wHcNNQea28LhwQ_amFamT33d5woVrJfJy53Z1k6V090/edit?usp=sharing).

![enter image description here](https://rawgit.com/ivancuric/js-perf/master/Screen%20Shot%202017-11-14%20at%2015.58.55.png)

Ouch.  While this image only covers the parse and compile time of the JS (more on that later) and not general performance, it's strongly correallated and can be treated as an indicator of general JS performance.

To quote Bruce Lawson, it['s the World-Wide Web, not the Wealthy Western Web](https://vimeo.com/194968584). So, your target for web performance is a device that's **~25x slower** than your MacBook or iPhone. Let that sink in for a bit. But it gets worse. Let's see what we're actually aiming for.


## What exactly is performant JS code?

Now that we know what our target platform is, we can answer the next question: what _is_ performant JS code? 

While there's no absolute classification of what defines performant code, we do have a user-centric performance model we can use as a reference: [The RAIL model](https://developers.google.com/web/fundamentals/performance/rail).

![enter image description here](https://raw.githubusercontent.com/ivancuric/js-perf/master/rail.png)


### Respond

If your app responds to a user action in under 100ms, the user percieves the response as immediate. This applies to tappable elements, but not when scrolling or dragging.

### Animate

On a 60Hz monitor, we want to target a constant 60 frames per second when animating and scrolling. That results in around 16ms per frame. Out of that 16ms budget, you realistically have 8-10ms to do all the work, the rest is taken up by the browser internals and other variances.

### Idle work

If you have an expensive, continuously running task, make sure to slice it into smaller chunks to allow the main thread to react to user inputs. You should not have a task that delays user input for more than 50ms.

### Load

You should target a page load in under 1000ms. Anything over, and your users start getting twitchy. This is a pretty difficult goal to reach on mobile devices as it relates to the page being interactive, not just having it painted on screen and scrollable. In practice it's even less:

![enter image description here](https://raw.githubusercontent.com/ivancuric/js-perf/master/1000ms.png)

In practice, aim for the 5s time to interactive mark. It's what Chrome uses in their [Lighthouse audit](https://developers.google.com/web/tools/lighthouse/audits/time-to-interactive).

Now that we know the metrics, [let's have a look at some of the statistics](https://storage.googleapis.com/doubleclick-prod/documents/The_Need_for_Mobile_Speed_-_FINAL.pdf):

 - 53% of visits are abandoned if a mobile site takes more than three seconds to load
 - 1 out of 2 people expect a page to load in less than 2 seconds
 - 77% of mobile sites take longer than 10 seconds to load on 3G networks
 - 19 seconds is the average load time for mobile sites on 3G networks

[And a bit more, courtesy of Addy Osmani](https://medium.com/reloading/javascript-start-up-performance-69200f43b201):

- Apps became interactive in 8 seconds on desktop (using cable) and 16 seconds on mobile (Moto G4 over 3G)
- At the median, developers shipped 410KB of gzipped JS for their pages

Feeling sufficiently frustrated? Good. Let's get to work and fix the web. ✊


## Context is everything 

You might have noticed that the main bottleneck is the time it takes to load up your website. Specifically, the JavaScript download, parse, compile and execution time. There's no way around it but to load less JavaScript and load smarter.

But what about the actual work that your code does aside from just booting up the website? There has to be some performance gains there, right?

Before you dive into optimizing your code consider what you are building. Are you building a framework or a VDOM library? Does your code need to do thousands of operations per second? Are you doing a time-critical library for handling user input and/or animations? If not, you may want to shift your time and energy somewhere more impactful.

It's not that writing performant code doesn't matter, but it usually makes little to no impact in the grand scheme of things, especially when talking about microoptimizations. So, before you get into a Stack Overflow argument about `.map` vs `.forEach` vs `for` loops by comparing results from JSperf.com, make sure to see the forest for the trees. 50k ops/s might sound 50× better than 1k ops/s on paper, but it won't make a difference in most cases. 

## Parsing, compiling and executing

Fundamentally, the problem of most non-performant JS is not running the code itself, but all the steps that have to be taken _before_ the code even starts executing.

We're talking about levels of abstraction here. The CPU in your computer runs machine code. Most programs you are running... Actually, considering the number of Electron apps these days... hmmm... For the sake of argument, let's say that most of the code you're running on your computer is in the compiled binary format. Meaning, all the OS-level abstractions aside, it runs natively on your hardware, no prep-work needed.

JavaScript is not pre-compiled. It arrives (via a relatively slow network) as readable code in your browser which is, for all intents and purposes, the "OS" for your JS program.

That code first needs to be parsed, that is, read and turned into an computer-indexable structure that can be used for compiling. It then gets compiled into bytecode and finally machine code, before it can be executed by your device / browser.

Another _very_ important thing to mention is that JavaScript is single-threaded, and runs on the browser's main thread. Meaning that only one process can run at a time. If your dev tools performance timeline is filled with yellow peaks, running your CPU at 100%, you'll have long / dropped frames, janky scrolling and all other kind of nasty stuff.

So there's all this work that needs to be done before your JS starts working. Parsing and compiling takes up to 50% of the total time of JS execution in Chrome's V8 engine.

![enter image description here](https://cdn-images-1.medium.com/max/2000/0*M94-AavlZjGoudZG.)

There are two things I want you to take away from this section.
1. While not neccessarily linearly, JS parse time scales with the bundle size. The less JS you ship, the better.
2. Every JS framework (React, Vue, Angular, Preact...) you use is another level of abstraction (unless it's a precompiled one, like [Svelte](https://github.com/sveltejs/svelte)). Not only will it increase your bundle size, but also slow down your code since you're not talking directly to the browser.

There are ways to mitigate this, such as using service workers to do jobs in the background and on another thread, using asm.js to write code that is more easily compiled to machine instructions, but that's a whole another topic.

What you can do however is to avoid using JS animation frameworks for everything and [read up on what triggers paints and layouts](https://csstriggers.com/). Use the libraries only when there is absolutely no way to implement the animation using regular CSS transitions and animations.

Even though they may be using CSS transitions, composited properties and `requestAnimationFrame()`, they are still running in JS, on the main thread. They're basically just hammering your DOM with inline styles every 16ms, since there's not much else they can do. You need to make sure that all your JS will be done executing in <8ms per frame in order to keep the animations smooth.

CSS animations and transitions on the other hand are running off the main thread — on the GPU if implemented performantly, without causing relayouts/reflows. 

Considering that most animations are running either during loading or user interaction, this can give your web apps the much-needed room to breathe.

The [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) is an upcoming feature set that will allow you to do performant JS animations off the main thread, but for now, stick to CSS transtitions and techniques like [FLIP](https://aerotwist.com/blog/flip-your-animations/).


## Bundle sizes are everything

Today it's all about bundles. Gone are the times of bower and dozens of `<script>` tags behind the closing `</body>` tag.

Now it's all about `npm install`-ing whatever shiny new toy you find on NPM, bundling them together with Webpack in a huge single 1mb JS file and hammering your users' browser to a crawl while capping off their data plans.

Try shipping less JS. You might not need [the entire lodash library](https://surma.github.io/underdash/) for your project. Do you absolutely _need_ to use a JS framework? If yes, have you considered using something other than React, such as [Preact](https://github.com/developit/preact) or [HyperHTML](https://viperhtml.js.org/), which are less than 1/20 the size of React? Do you need [TweenMax](https://greensock.com/tweenmax) for that scroll to top animation?

When you are done pruning the weeds and shipping less JS, try shipping it _smarter_. Ship what you need, when you need it. 

Webpack 3 has _amazing_ features called [code splitting](https://webpack.js.org/guides/code-splitting/) and [dynamic imports](https://webpack.js.org/guides/code-splitting/#dynamic-imports). Instead of bundling all your JS modules into a monolithic `app.js` bundle, it can automatically split the code using the `import()` syntax and load it asynchronously.

You don't need to use frameworks, components and client side routing to gain the benefit of it either. Let's say that you have a complex piece of code that powers your `.mega-widget`, which can be on any number of pages. You can simply write the following in your main JS file:
```js
if (document.querySelector('.mega-widget')) {
	import('./mega-widget');
}
```
If your app finds the widget on the page, it will dynamically load the required supporting code. Otherwise, all's good.
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzM4OTg2ODQyLDM1MjIzNTU5MywtNzY5NT
E1NDJdfQ==
-->