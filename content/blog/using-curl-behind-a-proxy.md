+++
date = "2013-11-15T22:47:03-07:00"
draft = false
title = "Using 'curl' behind a Proxy"

+++

This is a relatively stupid problem that took me a few seconds to figure out, but I figured I would post on it anyway. I have recently been working with [`createosxinstallpkg`](https://github.com/munki/createOSXinstallPkg), part of the tool set included to administer [Munki](https://github.com/munki/munki), an open source managed software update server (and client) for OS X machines.

I used `createosxinstallpkg` last year when I needed to build a never-booted upgrade package to for 10.7 clients to upgrade to 10.8.2, which was being certified in our enterprise environment at the time. Despite the fact that OS X Mavericks 10.9 is offered to users as a free upgrade, we have several users who feel more comfortable letting IT perform the work, so I use `createosxinstallpkg` to create the never-booted upgrade package our techs use to perform the "supervised" upgrades.

In our environment, we have a proxy server for all outgoing traffic to the public internet. I noticed that during the process to check against Apple's software update catalog for the location of the incompatible applications package file, that the `createosxinstallpkg` job was failing after not being able to hit `swscan.apple.com:443` and ultimately timing out. After some inspection of the `createosxinstallpkg` source code, I saw that the python script is actually performing a `curl` against the software update catalog and subsequent location to find the incompatible applications package. I knew that I was going to have to specify to curl that I was behind a proxy, either by editing the source or by providing environment variables. When I chose to just specify the environment variables, I used the command:

```
export https_proxy=https://internet.proxyserver.mycompany.com:3128/
```

Soon after entering and having the package creation fail again, I realized I did not correctly specify my credentials to authenticate to our proxy server for https traffic. I edited my command and subsequent environment variable per below:

```
export https_proxy=https://username:password@internet.proxyserver.mycompany.com:3128/
```

Great, now my syntax is working for me--I'm going to be golden 🏆. Then I realized I was editing environment variables for `curl` to use on my local administrator's environment variable file. Since `createosxinstallpkg` runs as the root user, I needed to make this change to the root user's environment variable instead.

Once I realized the err of my ways, and made the correct environment variable edit for the root user, `createosxinstallpkg` was able to successfully communicate for its files. Silly problem, but I wanted to document it anyway in case someone starts banging their head like I did for a second.
