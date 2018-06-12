---
layout: post
title: How to keep a command running despite closing the SSH connection
---
I'm sure it happened to you, more than once: you run a command on a remote host, thinking it will finish in a few seconds. Unfortunately, after 5 minutes looking at your terminal and hoping for the command to be done soon, you realize that it will take hours. Obviously, it's Friday afternoon and you had planned a trip for the weekend.

There are, I guess, two kind of persons in this situation:
- the first group will most likely SIGINT the command and postpone it to Monday morning;
- the second one will try to gracefully handle this situation by letting the command run and enjoying that well-deserved weekend trip.

You better leave here if you feel like you're a first-group person 😉

In case you'd like to let the command run even after closing the SSH connection, there is a solution, that I experienced myself recently (on a pg_dump, to be precise):
1. Send a <b>SIGTSTP</b> to the command by typing <b>CTRL+Z</b> in the terminal running your command. That will put the command in the background, in a "paused" state.
2. Run the <b>bg</b> command. This will resume your paused command, and most importantly, will run it in the background. We're getting close!
3. Last, run the <b>disown</b> command. This will remove (or let's say detach) your command from the shell session.

You can now safely <b>exit</b>, close your laptop and go 🚀!

Here is a 3-lines summary if you should already been gone:
<pre>
$ CTRL+Z
$ bg
$ disown
</pre>

Not too hard, right?

And please, next time, do yourself a favor: use <b>nohup</b> or <b>screen</b> before launching any remote command!
