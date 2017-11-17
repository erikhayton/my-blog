---
layout: post
title: How to rotate your logs hourly
---
A common practice with Docker containers is to log everything they output to `/dev/stdout` and `/dev/stderr`. This is perfectly fine for most containers. But with dockerized load-balancers, I can create issues. This is what we recently faced at Hunter.

On Ubuntu, the default `logrotate` behaviour is to rotate files weekly and keep 4 weeks of log. Now imagine a load-balancer serving millions of requests a day? That makes a lot of logs. As a consequence, the first issue we had was that the weekly log rotation was consuming *a lot* of CPU, at a point where the load balancer had troubles doing his load-balancing job. Scary. The second one, obviously, was about disk space. Keeping one month of requests, even gzipped, takes a lot of space.

Hopefully, I came up with a simple and obvious solution that fixed both problems: rotate the logs more often and keep them less long. If you want to do the same, it's a simple as:

1/ Create a new logrotate rule for your containers (`/etc/logrotate.d/containers` for example):

<pre>
/var/lib/docker/containers/*/*.log {
  rotate 24
  hourly
  compress
  size=10M
  missingok
  delaycompress
  copytruncate
}
</pre>

Here we'll rotate our logs hourly, and will keep only 24 of them. So that means we only keep the logs of the last 24 hours. Feel free to increase this at your convenience.

2/ Move `logrotate` from `/etc/cron.daily` (default location) to `/etc/cron.hourly`:
```
mv /etc/cron.daily/logrotate /etc/cron.hourly/logrotate
```

Now, your logs will be rotated on a hourly basis so you won't even notice the CPU usage increase during the log rotation. And as you'll keep only a few of them, your disk will be happy :)
