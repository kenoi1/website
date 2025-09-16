---
title: "Karton Virtual Machine Manager Blog #4: Hardware Accelerating the SPICE viewer (with OpenGL)"
date: "2025-09-15"
authors:
 - kenoi
tags:
cover:
    image: "/karton4-1.png"

SPDX-License-Identifier: CC-BY-SA-4.0
SPDX-FileCopyrightText: 2025 Derek Lin <derekhongdalin@gmail.com>
---

I was at [Akademy 2025](https://akademy.kde.org/2025/) last-last week and did the some preliminary research on optimizing the VM viewer on Karton. After some more work this past week, it's working! I'm still finishing up [the merge request](https://invent.kde.org/sitter/karton/-/merge_requests/33), but exciting news to come!

This has been something I've been planning on for quite a while now and will significantly improve the experience using Karton :) 

{{< video src="https://kenoi.dev/blogs/2025-09-15/compareviewr.mp4" >}}

*a comparison with an old video I had.*

*left: old viewer, right: new viewer*

## Old Rendering Pipeline

My original approach for rendering listened to `display-primary-create` and `invalidate-display-primary` [SPICE signals](https://www.spice-space.org/api/spice-gtk/SpiceDisplayChannel.html#SpiceDisplayChannel-display-invalidate). Everytime it received a callback, it would create a new QImage and render that to the QQuickItem (the viewer window). As you can imagine, this was very inefficient as it is basically generating new images for every single frame being rendered. It suffered a lot from screen-tearing any time there were sudden changes to the screen.

You can read more about it in [my SPICE client blog](https://kenoi.dev/blogs/2025-07-04/).

## Let's Fix It!

I had known about gl properties in SPICE for a while, but I kept putting it off since I really didn't want to deal with any more graphics stuff after the last attempt.

Fast forward to last-last week, I was attending my first ever KDE Akademy in Berlin and all of a sudden gained some motivation. It was really exciting hearing talks about all the *kool* things happening in KDE. 

<img src="https://kenoi.dev/blogs/2025-09-15/akademy.png" style="display: inline-block;" />

### gl-draw

My first order of business was getting the `gl-draw` signal to properly receive gl-scanouts from my SPICE connection. After setting up the callback, I found out that I had to reconfigure my VMs to properly support it.

This was easy enough as I've made the Karton VM installation classes a few months ago done through the libvirt domain XML format. VMs need enabling gl and 3d acceleration through the graphics element. The socket connection to SPICE also had to be switched from TCP to UNIX, which was set to `/tmp/spice-vm{uuid}.sock`. As a result, previous VMs configured in Karton should no longer work as the previous rendering pipeline has been removed.

Once properly configured, I was able to get `SpiceGlScanout` objects from my callback linked to the `gl-draw` signal. Now, I needed to render these scanouts onto my QQuickItem canvas.

### EGL stuff

Having no background in graphics, I pretty much had no idea what I was doing at this point. 

The SpiceGlScanout is a struct that looks like this:
```cpp
struct SpiceGlScanout {
    gint fd;
    guint32 width;
    guint32 height;
    guint32 stride;
    guint32 format;
    gboolean y0top;
};
```
The width, height, stride, etc..., are all parameters that can be used to set your final rendered frame, but the important field is the fd (file descriptor) which is a ["a drm DMABUF file that can be imported with eglCreateImageKHR"](https://www.spice-space.org/api/spice-gtk/SpiceDisplayChannel.html#SpiceDisplayChannel--gl-scanout). I didn't know what that was;  but at least I learned I should be using the EGL library to do the processing. 

I had found some forum articles ([Qt forum](https://forum.qt.io/topic/115805/render-eglimagekhr-into-qopenglframebufferobject/2), [Arm developer forum](https://developer.arm.com/documentation/ka004859/latest/)) related to rendering OpenGL textures which used the EGL library and were quite helpful. I also looked at the SPICE GTK widget source code which gave me some ideas on the GL parameters to work with. 

From these references, I saw that they pretty much followed the same pattern. Very simply put: 
```
-> create egl image from a bunch of attributes/settings
-> generate texture from the fd 
-> bind texture to a texture type 
-> "glEGLImageTargetTexture2DOES" use this function?? still don't know what this does lol
-> destroy egl image
```

I originally tried setting the GL context properties manually, but there were some issues with getting it to detect my display and thread syncronization. Then, I found out that Qt had a QOpenGLFunctions library which had all of the EGL functions and context properties wrapped and made my life a whole bunch easier.


### OpenGL texture -> Qt 
After a ton of trial and error, it looked like my EGL images were properly being created. Now I needed to render these GL textures to the QQuickItem. 

How you do so is, within the inherited `updatePaintNode()` function, you return a `QSGNode` which has the information for updating that frame. Looking through the Qt documentation, `QNativeTexture` is a struct that allows you to store a texture ID to an OpenGL image. With that, you can create a wrapper `QRhi` class with some of the generic context of the display. 

Finally, you can use the `createTextureFromRhiTexture()` function under QQuickWindow which allows you to create a QSGTexture from an RHI for a QSGNode that can be returned by `updatePaintNode()`. And, we're done! Yay!

To sum up, the framebuffer pipeline: 

`gl-draw signal->gl-scanout->import GL texture->GL texture ID->QNativeTexture->QRhi->QSGTexture->QSGNode->QQuickItem`
{{< video src="https://kenoi.dev/blogs/2025-09-15/rendered.mp4" >}}

*so much smoother! yes, I was very excited.*

### Socials

Website: https://kenoi.dev/

Mastodon: https://mastodon.social/@kenoi

GitLab: https://invent.kde.org/kenoi

GitHub: https://github.com/kenoi1

Matrix: @kenoi:matrix.org

Discord: kenyoy