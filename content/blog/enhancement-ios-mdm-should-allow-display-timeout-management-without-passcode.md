+++
date = "2017-02-17T02:01:00-07:00"
draft = false
title = "Enhancement: iOS MDM should allow display timeout management without passcode"
tags = ["iOS", "MDM", "Configuration", "Apple"]
categories = ["iOS", "MDM", "Configuration", "Apple"]

+++

Hey all!

I've just opened [rdar://30582319](http://www.openradar.me/30582319) in response to some testing I've been doing to attempt to manage the auto-lock settings on various iOS devices used as kiosks. In our case, we want to be able to set the display timeout to '15 minutes' or ideally 'Never', but not require the device to have a passcode enabled.

I've been testing this with a custom config profile, of which I've created a [gist here](https://gist.github.com/loyaltyarm/16deb0866697c3f2e3e33784453a393c).

Some interesting additional behavior I've seen on iOS 9.3 and iOS 10.x is also found in the radar ticket. I've seen two problem behaviors:

- the iOS device requires a passcode despite the `forcePIN` key being set to False.
- the iOS device does not recognize the updated integer under the `maxInactivity` key.

Ideally I'd like to see display timeout management broken out separately from passcode policy, or have the device/OS recognize that the `forcePIN` key is set to false and not require a passcode when the profile is installed.

If anyone has been able to manage display lock separately from the passcode I'd be interested in hearing about your success.
