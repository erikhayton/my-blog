---
layout: post
title: Setup a Heroku-like push to production mechanism
---
Heroku is a amazing Platform as a Service that enables you to build and run application in the cloud. It's often the first choice for deploying an app when you don't want to focus on something else that the app itself.

One great feature of Heroku is their "push to production" mechanism, which allows you to trigger deployments from your local environment itself. This removes the need from a CI/CD pipeline but keeps the simplicity and security of an automated deployment workflow. Therefore, this is great for small or side-projects.

This feature is easy to setup outside of Heroku, on any server with Git installed. Let's see how to do so!

<h3>A. On the server side</h3>

1/ Make sure Git is installed on the server. Else install it 😉

2/ Create the bare repository on your server (here's a [great resource](http://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/) to know more about the difference between work respositories and bare repositories):
<pre>
$ mkdir -p ~/git/app.git
$ cd ~/git/app.git
$ git init --bare
</pre>
This is where your code will be pushed to, but that's not the code your web server will execute. So I recommend to create the repository outside of your web "scope".

3/ Create the directory that will contain your app's code (the one executed by your web server):
<pre>
$ mkdir -p ~/www/app
</pre>

4/ Create a `post-receive hook`. That is where the real magic of all of it lies! This code will be executed every time you push code to that repository. In this post, we simply want to overwrite all the files in the web server with the ones in the repository, so here is how to handle it:
<pre>
$ nano ~/git/app.git/hooks/post-receive

#!/bin/bash

export GIT_WORK_TREE=/home/bastien/www/app
git checkout -f

$ chmod +x ~/git/app.git/hooks/post-receive
</pre>

Everything's ready on the server side, let's wrap everything up on the client side!

<h3>B. On the client side</h3>

1/ Add that new bare repository as a remote to your local repository:
<pre>
$ git remote add production
ssh://bastien@IP/home/bastien/git/app.git
</pre>

2/ Specify the ref as well (on your first push only):
<pre>
$ git push production +master:refs/heads/master
</pre>

3/ To deploy to production, you now simply need to type:
<pre>
$ git push production
</pre>

And all your latest work will be pushed to production automatically. No need for FTP, SSH or any custom deployment trick!

<h3>C. Conclusion</h3>

You might need to make advanced operations before deploying a new version of your code: generating assets, restarting the web server, etc. But as our post-receive script is a bash script, that shouldn't be too hard!

Happy deployments 😊
