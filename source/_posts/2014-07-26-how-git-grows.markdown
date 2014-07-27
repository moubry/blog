---
layout: post
title: "How Git Grows"
date: 2014-07-26 21:39:58 -0500
comments: true
categories:
---

<img src="/images/little-shop-of-horrors.jpg" alt="If you feed me, Seymour, I can grow up big and strong." width="100%">

The more you commit to a Git repository, the bigger the repository gets.

A project’s repository lives in a `.git` folder at its root. Our Rails project at TripCase has a 60M Git folder:

	$ du -sh ~/sabre/tripcase-rails/.git
	 60M	/Users/sean/sabre/tripcase-rails/.git

This folder could’ve been a lot bigger, but Git has [clever compression mechanisms to *only store the diffs*][packfiles] between files instead of the full file with every commit.

However, binaries can’t be diffed by Git. If you’re committing a lot of these files, even if the changes aren’t great, your repository can grow quite large.

An Exercise
-----------

Create a new repository:

	$ mkdir -p ~/Developer/backgrounds
	$ cd ~/Developer/backgrounds
	$ git init

Let’s make our first commit—we’ll throw in an image from elsewhere on the disk. Let’s find a good one:

	$ ls -lahS /Library/Desktop\ Pictures/

`Beach.jpg` looks good. We’ll move and rename it to `background.jpg`:

	$ cp /Library/Desktop\ Pictures/Beach.jpg background.jpg

Commit it:

	$ git add background.jpg
	$ git commit -am "Initial commit"

Now check out the [objects folder][packfiles]:

	$ du -sh ~/Developer/backgrounds/.git/objects/*
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/52
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/68
	10.0M	/Users/sean/Developer/backgrounds/.git/objects/c7
	  0B	/Users/sean/Developer/backgrounds/.git/objects/info
	  0B	/Users/sean/Developer/backgrounds/.git/objects/pack

Notice the "10.0M" object.

Let’s overwrite that file with a new image (the largest one in the backgrounds directory `Zebras.jpg`) and commit it:

	$ cp /Library/Desktop\ Pictures/Zebras.jpg background.jpg
	$ git commit -am "Zebras over Beaches"

The objects again:

	$ du -sh ~/Developer/backgrounds/.git/objects/*
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/27
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/52
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/68
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/7e
	10.0M	/Users/sean/Developer/backgrounds/.git/objects/c7
	 25M	/Users/sean/Developer/backgrounds/.git/objects/f0
	  0B	/Users/sean/Developer/backgrounds/.git/objects/info
	  0B	/Users/sean/Developer/backgrounds/.git/objects/pack

Use the app Preview to trivial change the content of the image. Add an arrow or some text somewhere on it. Leave it to be mostly the same image.

	$ open background.jpg

Commit the change:

	$ git commit -am "Adding an arrow to Zebras"

The objects again:

	$ du -sh ~/Developer/backgrounds/.git/objects/*
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/0f
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/27
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/52
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/68
	 28M	/Users/sean/Developer/backgrounds/.git/objects/7e
	10.0M	/Users/sean/Developer/backgrounds/.git/objects/c7
	 25M	/Users/sean/Developer/backgrounds/.git/objects/f0
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/fa
	  0B	/Users/sean/Developer/backgrounds/.git/objects/info
	  0B	/Users/sean/Developer/backgrounds/.git/objects/pack

There’s a new 28M object right there after the 25M object. Why? They’re essentially the same image, except for I added an arrow to it. Shouldn’t Git get be fancy and know how to only store the diff?

Let’s use the `git-gc` command to cleanup the `objects` directory:

	$ git gc
	Counting objects: 9, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (6/6), done.
	Writing objects: 100% (9/9), done.
	Total 9 (delta 0), reused 0 (delta 0)

	$ du -sh ~/Developer/backgrounds/.git/objects/*
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/info
	 63M	/Users/sean/Developer/backgrounds/.git/objects/pack

Oh, so that’s what cleanup does. The pack folder there contains the following files:

	$ du -sh ~/Developer/backgrounds/.git/objects/pack/*
	$ 4.0K	/Users/sean/Developer/backgrounds/.git/objects/pack/pack-ba1ac2ddcbb7e3d283a6eae112a073564bcdbdfd.idx
	$  63M	/Users/sean/Developer/backgrounds/.git/objects/pack/pack-ba1ac2ddcbb7e3d283a6eae112a073564bcdbdfd.pack

Notice that that pack file is 63M which is exactly the sum of the objects we had before (28M + 10M + 25M). Probably would’ve been more useful if there were unreachable objects or duplicate objects.

[According to documentation][git-gc], `git-gc` has an `--aggressive` option:

> Usually git gc runs very quickly while providing good disk space utilization and performance. This option will cause git gc to more aggressively optimize the repository at the expense of taking much more time. The effects of this optimization are persistent, so this option only needs to be used occasionally; every few hundred changesets or so.

Let’s see if that helps:

	$ git gc --aggressive
	Counting objects: 9, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (6/6), done.
	Writing objects: 100% (9/9), done.
	Total 9 (delta 0), reused 9 (delta 0)

	$ du -sh ~/Developer/backgrounds/.git/objects/*
	4.0K	/Users/sean/Developer/backgrounds/.git/objects/info
	 63M	/Users/sean/Developer/backgrounds/.git/objects/pack

Nope.

We Do This To Ourselves
-----------------------

GitHub has super useful, world-class image diffing support. In order to diff images, you have to commit multiple versions of the same image. And as we demonstrated above, Git isn’t great at optimizing for this behavior.

This isn’t the only feature GitHub supports despite it being a pretty bad idea. From their ["What is my disk quota?"][github-limit] page:

> GitHub supports rendering design files like PSDs and 3D models.

> Because these graphic file types can be very large, GitHub’s designers use a service like Dropbox to stay in sync. Only the final image assets are committed into our repositories.

GitHub isn’t alone:

* [Image diffing is popular with Git][diff]
* [Facebook’s visual regression testing tool Huxley encourages you to commit images to your repository][huxley]
* [Kaleidoscope heavily markets their image diffing features][kaleidoscope]

What We’ve Learned
------------------

Git has features that optimize disk space for text data. It’s not great at binary files, though.

Maybe it’s not technically possible to only store the diff from an image change. Maybe Git just doesn’t have this feature. Maybe committing images inefficiently isn’t that big of a deal.

Committing large binaries files is fine, I guess. The value seems to outweigh the costs, which appears to be performance, disk size, and clone time. Should be fine as long as you don’t project you’ll ever blow past your Git host’s disk quota. However, this seems like a hard thing to project.

Going back in time to remove binary files [does not look trivial][prune].

[Hit me up on Twitter][twitter] if you have any feedback.

[diff]: https://www.google.com/#q=image+diffing+gif
[github]: https://help.github.com/articles/rendering-and-diffing-images
[huxley]: https://github.com/facebook/huxley
[kaleidoscope]: http://www.kaleidoscopeapp.com
[packfiles]: http://git-scm.com/book/en/Git-Internals-Packfiles
[git-gc]: https://www.kernel.org/pub/software/scm/git/docs/git-gc.html
[github-limit]: https://help.github.com/articles/what-is-my-disk-quota
[twitter]: https://twitter.com/moubry
[prune]: http://naleid.com/blog/2012/01/17/finding-and-purging-big-files-from-git-history
