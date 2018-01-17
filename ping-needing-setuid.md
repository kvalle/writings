---
date: 2016-02-13
title: Ping requiring sudo‽
description: "The ping command suddenly started requiring sudo to run. This is why, and how to fix it."
---

## TL;DR:

*Problem*: `ping` is complaining of `ping: icmp open socket: Operation not permitted"`.  
*Solution*: Run `sudo chmod u+s /bin/ping` (or whatever `which ping` points to).

## What's going on

I encountered this bug today just after having installed Raspbian on my Raspberry Pi from [the latest NOOBS-image](https://www.raspberrypi.org/downloads/noobs/) (version 1.7).

```
pi@raspberrypi:~ $ ping example.com
ping: icmp open socket: Operation not permitted"
```

It was still working fine with `sudo`. It turns out that this is actually a known bug on this particular version of Raspbian, and the solution is to change the file permissions of `ping`.

Lets have a look on the before and after:

```
pi@raspberrypi:~ $ ls -la /bin/ping
-rwxr-xr-x 1 root root 38844 Feb 12  2014 /bin/ping
pi@raspberrypi:~ $ sudo chmod u+s /bin/ping
pi@raspberrypi:~ $ ls -la /bin/ping
-rwsr-xr-x 1 root root 38844 Feb 12  2014 /bin/ping
```

As you might see if you look closely, the priveliges for *user* is `rwx` rather than `rws`.

This is a file permission I'd never used before, so, as always, I headed to the `man` page for more details. Apparently `s` is the "set-user-ID-on-execution and set-group-ID-on-execution bits" (depending on whether it appears for *user* or *group*). They are called the *setuid* and *setgid* bits for short.

When these bits are on, executable files will run with the effective user ID (or group ID) set to those of the files owner. Since `ping` is owned by `root`, this means you're elevated to super user every time you ping something. (Which is quite neccessary, since you need to send and listen for control packets on a network interface).
