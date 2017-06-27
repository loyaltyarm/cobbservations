+++
date = "2016-10-27T22:03:31-07:00"
draft = false
title = "Fixing Persistent Notifications on iOS 10"

+++

After upgrading to iOS 10, quite a few people have noticed a rather annoying change in the behavior of notifications on their devices. In particular, banner-style notifications that were automatically dismissed in previous versions of iOS, seem to stay on the screen until an explicit action is taken to snooze or dismiss them. Well, if they seem that way, it's because that's how they were designed. In fact, notifications can carry three styles and have for quite some time on iOS:

1. Alerts - "require an action before proceeding"
2. Banners - "appear at the top of the screen and go away automatically"
3. None

What makes this slightly confusing on iOS 10 is that all notifications of either style 1 or 2 actually come through as banners, which is actually the real change in how notifications work on iOS. Since Apple made this change, alerts that were previously [modals](https://en.wikipedia.org/wiki/Modal_window) are now presented as banners and must be dismissed manually. The good news is that the fix is extremely easy!

I've included steps below to fix this for the Calendar app, since that's the app that was causing this on my device. Since I use only my personal calendar for non-important meetings, I only want to be alerted by my work calendar. You can follow the same steps below for other apps on your device as needed.

![notifications-1](/img/notifications-1.png)

![notifications-2](/img/notifications-2.png)

![notifications-3](/img/notifications-3.png)

![notifications-4](/img/notifications-4.png)

![notifications-5](/img/notifications-5.png)
