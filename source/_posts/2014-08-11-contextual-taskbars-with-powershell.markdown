---
layout: post
title: "Contextual Taskbars with Powershell"
author: Jami Moubry
date: 2014-08-11 21:54:28 -0500
comments: true
categories:
---

Problem
------------

I use my personal laptop (Windows 8) for a variety of tasks from perusing social networks to writing blogs to developing software. Each of these requires a completely different set of applications and bookmarks. I don't need Excel when I'm writing a blog, and I don't need Picasa when I'm developing an app.

I want my desktop to be tailored to the task at hand to promote focus. I also have more apps than will fit on my taskbar, but I like having them there to remind me of the tools I have available (out of sight out of mind).

<img src="/images/desktop_before.png" width="100%">

Solution
------------

Write a script to save and restore sets of taskbar icons.

I wanted to throw something together quickly and this script will only ever be useful on a Windows machine so I chose Powershell. I discovered that Windows persists taskbar data in two different locations.

The paths to the applications and application start settings are stored in the registry under:

    HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Taskband

Funny how they call it the taskband in the registry but everywhere else it's the taskbar. The actual icons are stored in:

    C:\Users\$username\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar

This makes no sense to me--what does the taskbar have to do with Internet Explorer or Quick Launch? Windows Store Apps don't require an icon in that folder.

<img src="/images/taskbar_folder.png" width="100%">

The script I ended up with uses a Windows Forms interface (remember I said I wanted something *quick*, not pretty) and basically just exports and imports the taskbar registry/icons. It uses the `reg` command to export the taskband registry to a file and it copies the icons into a folder and then import does the reverse. [View the script on GitHub](https://github.com/jmoubry/context-taskbar).

<img src="/images/script_output.png">

I also added a shortcut to the taskbar that runs my script as an administrator when you click on it so that I can quickly switch between taskbar configurations:

<img src="/images/taskbar_helper_icon.png">

__Blogging Desktop__

<img src="/images/desktop_after_blogging.png" width="100%">

__Web Development Desktop__

<img src="/images/desktop_after_webdevelopment.png" width="100%">


Improvements
------------

When I have time, these are the next steps for the script:

* Improve on the UI by switching to WPF--there are several Powershell modules out there to help with that.
* Generate a visual overview of what each of the taskbar contexts that have been saved (exported) look like.
* Launch Chrome with some bookmarks pre-opened based on the selected context.
