Operations of Moubry Blog
=========================

Installing
----------

	git clone https://github.com/moubry/blog.git
	git remote add octopress https://github.com/imathis/octopress.git

Updating
--------

Get the latest from Brandon Mathis:

	git pull --rebase octopress master

Force it up to your personal remote:

	git push --force origin master

Posting
-------

	rake new_post["Zombie Ninjas Attack: A survivor's retrospective"]

Deploying
---------

	rake gen_deploy

For reference, this executes the following Rake task:

	desc "Generate website and deploy"
	task :gen_deploy => [:integrate, :generate, :deploy] do
	end
