---
title: "Karton Virtual Machine Manager GSoC 2025 Blog #2: Qt SPICE Client"
date: "2025-07-04"
authors:
 - kenoi
draft: false
tags:
- kde
- codes
cover:
    image: "/minekarton.png"
    caption: "new logo for Karton! hmmm wait a moment..."
SPDX-License-Identifier: CC-BY-SA-4.0
SPDX-FileCopyrightText: 2025 Derek Lin <derekhongdalin@gmail.com>
---

After my [initial status blog](https://blogs.kde.org/2025/05/18/gsoc-2025-project-intro-developing-karton-the-kde-virtual-machine-manager/), I was really surprised to see so much support and excitement about [Karton](https://invent.kde.org/sitter/karton), and I'm grateful for it!

A few weeks have gone by since the official coding period for Google Summer of Code began. I wanted to share what I've been working on with the project!

## VM Installer

Earlier last month, I was finishing up addressing feedback (big thanks again to [Harald](https://invent.kde.org/sitter)) on the [VM installer-related MR](https://invent.kde.org/sitter/karton/-/merge_requests/8). I had made some improvements to memory management, bug fixes related to detecting ISO disks, as well as refactoring of the class structures. I also ported it over to using [QML modules](https://api.kde.org/ecm/module/ECMQmlModule.html), which is much more commonly used in KDE apps, instead of exposing objects at runtime.

After a bit more review, this has now been merged into the master branch! This was what was featured in the previous demo video and you can find a list of the full changes on the commit.

## SPICE Client

Two weeks ago, I started to get back to work on my [SPICE viewer branch](https://invent.kde.org/sitter/karton/-/merge_requests/15). This is the main component I had planned for this summer. 

It took a few days to clean up my code that connects to the SPICE display and inputs channel.

However, a lot of my time was spent trying to get a properly working frame buffer that grabs the VM display from [SPICE (spice-client-glib)](https://www.spice-space.org/spice-gtk.html) and renders it to a native KDE window. The approach I originally took was rendering the pixel array I received to a QImage which could be drawn onto a QQuickItem to be displayed on the window. It listens to SPICE callbacks to know when to update, and was pretty exciting to see it rendering for the first time!

One of the most confusing issues I encountered was when I was encountering weird colour and transparency artifacts in my rendering. I initially thought it was a problem due to the QImage 32-bit RGB format I was labelling the data as, so I ended up going through [a bunch of formats](https://doc.qt.io/qt-6/qimage.html#Format-enum) on the Qt documentation. The results were very inconsistent and 24-bit formats were somehow looking better, despite SPICE giving me it in 32-bit. Turns out (unrelatedly), there was some race condition with how I was reading the array while SPICE was writing to it, so manually copying the pixels over to a separate array did the trick.

Here are nice pictures from my adventures!

<div style="text-align: center;">
    <img src="https://kenoi.dev/blogs/2025-07-04/noice.png" width="400" style="display: inline-block;" />
    <img src="https://kenoi.dev/blogs/2025-07-04/nnice.png" width="500" style="display: inline-block;" />
</div>

*my first time properly seeing the display... (also in the wrong format)* ( •͈ ૦ •͈ )

I have also set up forwarding controls which listens to Qt user input (mouse clicks, hover, keyboard presses) and maps coordinates and events to [SPICE messages](https://www.spice-space.org/spice-protocol.html) in the inputs channel. Unfortunately, Qt key event scancodes seem to be in evdev format while SPICE expects PC XT. Currently, I have been manually mapping each scancode, but I might see if I can switch to use some library eventually.

Once I polish this up, I hoping to merge this into master soon. It'll likely be very slow and barebones, but I'm hoping I can make more improvements later on!

{{< video src="https://kenoi.dev/blogs/2025-07-04/viewer.mp4" muted="true" loop="true" >}}

*still very lagging scrolling, but now we can read [Pepper & Carrot](https://www.peppercarrot.com/)!*

## What's Next?

While relatively simple, I noticed my approach is quite inefficient, as it has to convert every received frame to a QImage, and suffers from tearing when it has to update quickly (ex: scrolling, videos). 

SPICE has a [gl-scanout property](https://www.spice-space.org/api/spice-gtk/SpiceDisplayChannel.html#SpiceGlScanout) which is likely much more optimized for rendering frames and I plan on looking into switching over to that in the long-term. 

I also need to implement audio forwarding, sending proper mouse drag events, and resizing the viewing window.

On a side note, I also helped review a nice [QoL feature from Vishal](https://invent.kde.org/sitter/karton/-/merge_requests/16) to list the OS variants in the installation dialog. I've just been memorizing them up until now... :')

Hopefully, once I get the SPICE viewer to a reasonable state, I can get back to improving the installation experience further like adding a page to download ISOs from.

As I mentioned a bit previously, I also want to rework the UI eventually. This means spending time to redevelop the components to include a sidebar, which is inspired by [UTM](https://mac.getutm.app/) and [DistroShelf](https://nginx-flathub.apps.openshift.gnome.org/lt/apps/com.ranfdev.DistroShelf).

## Lastly,

I also wanted to make a bit of a note on my plans and hopes throughout the GSoC period. After working on developing these different components of the app, I started to realize how much time goes into polishing, so I believe that I need to prioritse some of the most important features and making them work well.

Overall, it's been super busy (since I'm also balancing school work), but it has been quite exciting!

Come join our matrix channel: [karton:kde.org](https://matrix.to/#/#karton:kde.org)

Another thing, I recently made a personal website, [kenoi.dev](https://kenoi.dev/), where I also plan on blogging! 

That's all, thank you for reading :D