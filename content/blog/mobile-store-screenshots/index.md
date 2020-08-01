---
title: "Mobile Store Screenshots"
date: 2020-07-30T23:54:32+02:00
author: "Evert van Nieuwenburg"
draft: true
layout: "post"
mathjax: false
---

### 1\. The Problem
If you've ever published an app on a mobile store like Google's Play Store or Apple's App Store, you'll have gone through
the trouble of preparing mandatory screenshots at god knows how many resolutions. On the App Store, for example, you'll need
screenshots for 5.5" devices, 6.5" devices and ...

Now you could --theoretically-- run your game in a simulator a few times and take screenshots. But how will you reproduce the
same screenshot every time for the different resolutions? A slightly better solution would be using Unity's built in
ScreenCapture.CaptureScreenshot function, and change the resolution of the Gameview while your game is paused.
But that still requires manually changing the resolution a few times to gather all the sizes you need. Wouldn't it be easier
if you could gather all the screenshots you'll ever need through the press of a single button?

### 2\. The Solution
Well, such a button exists, and it comes with a keyboard shortcut to boot! All you need to do to get access to this
button is add the script I'm about to link to to your project, and put it on an active GameObject in your scene (might I suggest
just putting it on the main camera?). This solution works with postprocessing, with overlay and world camera's, and has a customizable list of resolutions that you
can set.

If you're interested in how this script works, please read on. If you're just looking for the fastest way to get your screenshot-fix,
here is the script.

Don't forget to check out the reason we made this screenshot script: Guinea Pug! You would make us very happy
if you could let the world know about our game and it's groundbreaking screenshot tech!

### 3\. The Workings
