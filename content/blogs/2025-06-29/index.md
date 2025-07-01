---
title: "Karton GSoC 2025 Blog #2: VM Installer + work on Qt SPICE Client"
date: "2025-06-30"
authors:
 - kenoi
draft: true
tags:
- kde
- codes
cover:
    image: "/minekarton.png"
    caption: "Karton! wait..."
SPDX-License-Identifier: CC-BY-SA-4.0
SPDX-FileCopyrightText: 2025 Derek Lin <derekhongdalin@gmail.com>
---
WIP

A few weeks have gone by since the official coding period for Google Summer of Code began. I wanted to share what I've been happening with the project!

After my initial status blog, I was really surprised to see so much support and excitement about Karton, and I'm grateful for it!

## VM Installer

Earlier this month, I was finishing up addressing feedback on the VM installer-related MR. I had made some improvements to memory management, bug fixes related to detecting iso disks, as well as refactoring of the class structures. I also ported it over to using QML modules, which is more commonly used in KDE apps, instead of exposing objects at runtime.

After a bit more review, this has now been merged into the master branch! This was what was featured in the previous demo video and you can find a list of the full changes on the commit.

## SPICE Client

Two weeks ago, I started to get back to work on my SPICE viewer branch. This is the main feature I planned for this summer. 

I spent a lot of time trying to get a properly working frame buffer that grabs the VM display from SPICE and renders it to a native KDE window. The approach I took was rendering the pixel array I recieved to a QImage which could be drawn onto a QQuickItem to be displayed on the window. While it listens to SPICE callbacks to know when to draw, it noticed it still is quite inefficient and suffers from tearing when it has to update quickly (ex: videos). 

One of the most confusing issues I encountered was when I was encountering weird artifacts in my rendering. I initially thought it was a problem due to the QImage 32-bit RGB format I was labelling the data to, so I ended up going through almost every format on the Qt documentation. The results were very inconsistent and 24-bit formats were somehow looking better, despite SPICE telling me it was 32-bit. Turns out (unrelatedly), there was some race condition with how I was reading the array while SPICE was writing to it, so manually copying the pixels did the trick. Eventually, I think we will have to switch to using gl-scanout in order to do partial updates based on a region that has changed.

I had also originally wanted to have the viewer replace the VM manager's window, like GNOME Boxes had done, but I ended up deciding on a separate window for simplicity. 

I had also set up forwarding controls which listens to Qt user input (mouse clicks, hover, keyboard presses) and maps coordinates and events to SPICE controls. This was surprising easy to set up since they were quite compatible!

## Update on Plans

I also wanted to make a bit of a note on my plans and hopes throughout the GSoC period. After working on developing these different components of the app, I started to realize how much time goes into polishing, so I believe that I need to revise the scope of what I want to do to make sure I don't over-extend myself.

My main priority is definitely a super nice and easy installation experience. This means spending time to redevelop the UI to include a sidebar, likely similar to UTM and Distrobox.

Overall, making a Qt SPICE client has been quite exciting as this is the last step to becoming a "KDE native" VM manager! There is still so much work to do though 