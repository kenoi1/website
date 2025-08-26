---
title: "DRAFT: GSoC 2025 Final Project Blog: Developing Karton, the KDE Virtual Machine Manager!"
date: "2025-08-23"
authors:
 - kenoi
tags:
- codes
cover:
    image: "/konqi.png"

SPDX-License-Identifier: CC-BY-SA-4.0
SPDX-FileCopyrightText: 2025 Derek Lin <derekhongdalin@gmail.com>
---

#### **Note: This blog post is a draft and still being worked on until Sept. 1**

Hello again everyone!

 I'm Derek Lin also known as [kenoi](https://invent.kde.org/kenoi), a second-year Math student and the University of Waterloo.

Through Google Summer of Code (GSoC), mentored by [Harald Sitter](https://invent.kde.org/hsitter), [Tobias Fella](https://invent.kde.org/tfella), and [Nicolas Fella](https://invent.kde.org/nicolasfella), I have been developing Karton, a KDE-native Virtual Machine Manager.

As the program wraps up, I thought it would be a good idea to put together what I've been able to accomplish as well as my plans going forward.

<img src="https://kenoi.dev/blogs/2025-08-23/karton.png" width="400" style="display: inline-block;" />

*A final look at Karton after the GSoC period.*

## Research and Initial Work

The main motivation behind Karton is to provide KDE users with a more Qt-native alternative to GTK-based virtual machine managers, as well as an easy-to-use experience.

I had first expressed interest in working on Karton in early Feburary where I made the initial full rewrite, using libvirt and a new UI, wrapping `virt-install`and `virt-viewer` CLIs. During this time, I had been doing research, writing a proposal, and trying out different virtual machine managers like GNOME Boxes, virtmanager, and UTM.

You can read more about it in my [project introduction blog]()!

<img src="https://kenoi.dev/blogs/2025-08-23/list.png" width="400" style="display: inline-block;" />

*A screenshot of my rewrite in March 8, 2025.*

### VM Installation

One of my goals for the project was to develop a custom libvirt domain XML generator using Qt libraries and the `libosinfo` GLib API. I started working on the feature in advance in April and was able to have it ready for review before the official GSoC coding period.

I created a dialogue menu to accept a VM name, installation media, storage, allocated RAM, and CPUs. libosinfo will attempt to identify the file and return a OS short-ID (ex: fedora40, ubuntu24.04, etc), otherwise users will need to select one from the displayed list.

Through the OS ID, libosinfo can provide certain specifications needed in the libvirt domain XML. Karton then fills in the rest, generating a UUID, a MAC address, and configuring devices and ports. The XML file is assembled through QDomDocument and passed into a libvirt call that verifies it before adding the VM.

VM information in Karton is parsed explicitly from the saved libvirt XML file found in the libvirt QEMU folder.

All in all, this addition completely removed the virt-install dependency although barebones.

 <img src="https://kenoi.dev/blogs/2025-08-23/installationdialog.png" width="400" style="display: inline-block;" />

 *A screenshot of the VM installation dialog*
 
 The smooth and easy VM installation process of GNOME Boxes had been an inspiration for me and I'd like to improve it in the future by adding a media installer and better error handling later on.
 
## Official Coding Begins!

A few weeks into the official coding period, I had been addressing feedback and polishing my VM installer merge request. This introduced much cleaner class interface separation in regards to storing individual VM data.

### SPICE Client and Viewer

My use of `virt-viewer` previously was meant as a temporary addition, being poorly integrated in Qt/Kirigami and lacks needed customizability. 

 <img src="https://kenoi.dev/blogs/2025-08-23/virtviewer.png" width="400" style="display: inline-block;" />

 *Previously, clicking the `view` button would open a virtviewer window*

As such, the bulk of my time was spent working with SPICE directly in order to create a custom Qt SPICE client and viewer. This needed to manage the state of connection to VM displays and render them to KDE-native windows. Other features such as input forwarding, audio receiving also needed to be implemented. 

I had configured all Karton-created VMs to be set to autoport for graphics which dynamically assigns a port at runtime. Consequently, I needed to use a CLI tool (virsh domdisplay) to fetch the SPICE URI to establish the initial connection.

The viewer display works through a frame buffer. You can read more about the blog

<div style="text-align: center;">
    <img src="https://kenoi.dev/blogs/2025-07-04/noice.png" width="400" style="display: inline-block;" />
    <img src="https://kenoi.dev/blogs/2025-07-04/nnice.png" width="500" style="display: inline-block;" />
</div>

I had to manage receiving and forwarding Qt input. Sending QMouseEvents, mouse button clicks, were straightforward and can be mapped directly to SPICE protocol mouse messages when activated. Keystrokes are taken in as QKeyEvents and the recieved scancodes, in evdev, are converted on the PC XT for SPICE through a map generated by QEMU. Implementing scroll and drag followed similarly. 

I also needed manage receiving audio streams from a SPICE callback, writing to a QAudioSink. One thing I found nice is how my approach supported multiple SPICE connections quite nicely. For example, opening multiple VMs will create separate audio sources for each so users can modify volume levels accordingly.

Later on, I added display frame resizing when the user resizes the Karton window as well as a fullscreen button. I noticed that doing so still causes resolution to appear quite bad, so proper resizing done through the guest machine will have to be implemented in the future.

(video)

### UI

My final major MR was to rework my UI to make better use of screen space. I moved the existing VM ListView into a sidebar displaying only name, state, and OS id. The right side would then have the detailed information of the selected VM. One my biggest inspirations was MacOS UTM's screenshot of the lastv active frame.

When a user closes the Karton viewer window, the last frame is saved to `user/.local/state/KDE/Karton/previews`. Implementing cool features like these are much easier now that we have our own viewer! I also added some effects for opacity and hover animation to make it look nice.

(macos utm, compare with Karton video hovering over screencap)

 <div style="text-align: center;">
    <img src="https://kenoi.dev/blogs/2025-08-23/manager.png" width="400" height="250" style="display: inline-block;" />
    <img src="https://kenoi.dev/blogs/2025-08-23/utm.png" width="400" height="250" style="display: inline-block;" />
</div>



Finally, I worked on media disc ejection. This uses a libvirt call to simulate the installation media being removed from the VM, so users can boot into their virtual hard drive after installing. 

Working through MRs has given me a lot of valuable and relevant industry experience going forward. A big thank you to Harald Sitter who has been reviewing and providing feedback along the way!

## Basic Feature Overview

## Demonstration

As a final test of the project, I decided to configure, configure and use a Fedora KDE VM using Karton. After specifying specs, I installed it to the virtual disk, ejected the installation media, and properly booted into it. Then, I tried playing some games. Overall, it worked pretty well!

## List of MRs

#### Major changes:

- [#4 Complete rewrite with libvirt backend, new UI](https://invent.kde.org/sitter/karton/-/merge_requests/4)
- [#6 Implement disk path and proper deletion button behavior](https://invent.kde.org/sitter/karton/-/merge_requests/6)
- [#8 VM creation through libvirt domain xml format](https://invent.kde.org/sitter/karton/-/merge_requests/8)
- [#15 Custom Qt SPICE client and viewer](https://invent.kde.org/sitter/karton/-/merge_requests/15)
- [#25 Revamp UI with sidebar and VM preview screencap](https://invent.kde.org/sitter/karton/-/merge_requests/25)
- [#26 Implement eject ISO disk button](https://invent.kde.org/sitter/karton/-/merge_requests/26)

#### Subtle changes:
- [#10 Update stop VM button icon](https://invent.kde.org/sitter/karton/-/merge_requests/10)
- [#14 Store XML path of ~/.config/libvirt/qemu
](https://invent.kde.org/sitter/karton/-/merge_requests/14)
- [#17 Fix fullscreen button anchor margin](https://invent.kde.org/sitter/karton/-/merge_requests/17)
- [#22 Extract installation dialog into a new class](https://invent.kde.org/sitter/karton/-/merge_requests/22)
- [#23 Error notification/prevention for empty installation fields](https://invent.kde.org/sitter/karton/-/merge_requests/23)
- [#24 List OS variants through searchable combo box](https://invent.kde.org/sitter/karton/-/merge_requests/24)

## Difficulties

My biggest regret was having a study term over this period. There were times I had a lot of trouble managing my time, balancing studying, searching for job positions, and contributing. Though it's been an exhausting school term, I am still super glad to have been able to contribute to a really cool project and get something work!

I was also quite new to both C++ and Qt development. Funny enough, I had been taking, and struggling on, my first course in C++ while working on Karton. I also spent a lot of time reading documentation to familiarize myself with a lot of the different APIs (libspice, libvirt, and libosinfo)

## What's Next?

There is still so much to do! Currently, I am on vacation and I will be attending Akademy in Berlin in September so I won't be able to work much until then. In the fall, I will be finally off school for a 4 month internship (yay!!). I'm hoping I will have more time to contribute again.

There's still a lot left especially with regards to the viewer. 

Here's a bit of an unorganized list:
* Optimize VM display frame buffer with SPICE `gl-scanout`
* Improved scaling and text rendering
* File transfer and clipboard through SPICE
* Full VM snapshotting through libvirt (full duplication)
* Browse and installation tool for commonly installed ISOs through QEMU
* Error handling in installation process
* Configuration of existing VMs

## Release?

In its current state, Karton is not feature complete, and not ready for officially packaging and releasing. In addition to the missing features listed before, there have been a lot of new and moving parts throughout this coding period, and I'd like to have the chance to thorough test the code to prevent any major issues.

However, I do encourage you to try it out (at your own risk!) by cloning the repo. Let me know what you think and when you find any issues!

In other news, there are some discussions of packaging Karton as a Flatpak eventually and I will be requesting to add it to the KDE namespace in the coming months, so stay tuned!

