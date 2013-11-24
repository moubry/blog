---
layout: post
title: Why I'm Dumping Eclipse Forever
author: Jami Moubry
date: 2012-08-25 20:55
comments: true
categories:
---

During college we were strongly encouraged to use Eclipse for Java development. Even then, I felt like Eclipse was difficult to use, so I chose to use JCreator. Or maybe I just wanted to rebel, not wanting to take suggestions from the same people who forced me to use vi for C++ development. I guess I didn't have that appreciation that others got from memorizing vi commands and knowing that only a subset of people know how to use that text editor.

Fortunately, my career led me down the Microsoft path, so I never had to deal with Eclipse—until Android showed up.

Having an Android phone and being a software developer, of course I wanted to make apps for it. I decided to give Eclipse another try, maybe I had unfairly judged it in college, maybe it was better now, and all the tutorials were based on using Eclipse anyway. I developed the first version of [Worth Watching](https://play.google.com/store/apps/details?id=com.moubry.worthwatching) using Eclipse and had relatively few problems with the IDE. As time went on, I spent more time developing the app in Eclipse and naturally there were software updates to Eclipse and the Android tools. I began to see more and more problems. First, when debugging on a real device, the Log Cat would frequently stop showing logs, especially when I cleared the log screen, no more logs would appear. With Android, you have folders for layouts with common filenames among the folders. When I had multiple files open in Eclipse that had the same filename, I could double-click on a file in the navigator, and it would bring the wrong file into view. For a while, when I selected the AVD Manager from the file menu, it would open the Android SDK Manager instead. At least a couple of times, my projects would out-of-nowhere fail to compile. This is what ultimately led me to give up on Eclipse.

I was so frustrated with Eclipse, that I decided to use a text editor and command line to do my development. Shortly after choosing the best text editor for the job—Sublime Text—I discovered IntelliJ's Community Edition which supports Android development. After struggling with Eclipse for almost a year, I was continuously surprised by how easy IntelliJ was to use. Remember how my projects wouldn't compile in Eclipse? Well, IntelliJ was more than happy to build them without any changes. I had experimented with a few Git plug-ins for Eclipse, but they were even harder to use than Eclipse was. IntelliJ has Git build-in AND it was extremely easy to figure out.

Everyone should dump Eclipse. Even if you can't convince your team to switch, IntelliJ supports compatibility with Eclipse and even has an Export to Eclipse feature. If I'm ever locked into Eclipse because it's the only IDE for a certain product, I will definitely go command-line only.

Farewell, Eclipse!