---
layout: post
title: "TunTap Doesn’t Work on Yosemite"
author: Sean Moubry
date: 2014-10-21 10:06:06 -0500
comments: true
categories:
---

When I upgraded to Yosemite, I could no longer use `vpnc` to VPN into work. I got the following error:

    can't initialise tunnel interface: No such file or directory

If you Google “tuntap does not work on yosemite” you might find this blog post. You’ll also find [this issue on GitHub for the Homebrew project](https://github.com/Homebrew/homebrew/issues/31153). Which links to [this umbrella issue on the Homebrew project](https://github.com/Homebrew/homebrew/issues/31164).

## What’s wrong?

[TunTap](http://tuntaposx.sourceforge.net) (a dependency of the VPNC) does not work on Yosemite because OS X no longer allows unsigned kernel extensions.

Previous versions (like Mavericks) allowed unsigned kernel extensions to run.

The easiest solution to this issue is globally disabling the system security policy that requires kernel extensions to be signed.

## How to disable kext signing requirement

You should not do this. But if you’re like me, you need to VPN into work right now, and are willing to accept the risks. Because worst case scenario, it’s exactly as insecure as Mavericks, which you were using yesterday.

These instructions are adapted from [this](http://www.cindori.org/trim-enabler-and-yosemite/) and [this](https://gist.github.com/leomelzer/3931794).

Uninstall TunTap:

    brew unintall tuntap

Boot into recovery mode by holding Cmd+R during reboot. Open terminal and further ensure tuntap is gone:

    rm -rf /Library/Extensions/tap.kext
    rm -rf /Library/Extensions/tun.kext
    rm -rf /Library/StartupItems/tap
    rm -rf /Library/StartupItems/tun

List all of the existing `boot-args` you’ve set:

    nvram boot-args

Does it say “kext-dev-mode=1”? If so, just restart because you’re already good!

Disable required signing of kernel extensions:

    nvram boot-args="-v kext-dev-mode=1"

Restart.

Install TunTap:

    brew install tuntap

Run the commands listed in Homebrew/TunTap’s installation output.

Start TunTap by loading its kernel extensions:

    sudo kextload /Library/Extensions/tap.kext
    sudo kextload /Library/Extensions/tun.kext

Verify by running `ls /dev/tun*` and confirm that it lists ~10 virtual interfaces.

If you ever get the `can't initialise tunnel interface` error again, check to see if TunTap’s kexts are loaded by running the above command again. You may need to configure OS X to load them at startup.

## Undo

Boot into recovery mode.

Remove all arguments:

    nvram -d boot-args

Restart.
