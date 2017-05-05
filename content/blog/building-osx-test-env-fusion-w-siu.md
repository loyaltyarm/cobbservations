+++
title = "Building OS X Test Environments in VMWare Fusion w/ System Image Utility"
draft = false
date = "2013-06-11T01:29:08-07:00"


+++

A while ago, Rich Trouton posted a method for setting up an [easily maintainable OSX virtual machine environment for testing](https://derflounder.wordpress.com/2013/01/23/building-mac-test-environments-with-vmware-fusion-netboot-and-deploystudio/), using DeployStudio (DSS) and a few other tools. His method is great because it allows you to build a specific test image (base) and store that as a workflow and master pair in your DSS environment. As Rich showed, it then becomes very easy to get going quickly when you need a fresh virtual machine to test new applications or other scenarios.

With today’s release of OS X Mavericks at WWDC, I found myself wanting to test a few things from the Developer beta(s) in a virtual machine but quickly realized I hadn’t really setup the ideal environment Rich describes in my home test lab, despite having the necessary equipment. I thought about running through my home DSS setup to get a test VM workflow setup, but due to the somewhat ‘temporary’ status of the developer beta, I didn’t want to spend too much time setting things up–what with [the coming winter](https://www.forbes.com/sites/erikkain/2013/06/10/game-of-thrones-season-3-finale-review-winter-is-coming/#559f63364008) and [impending alien attacks](http://www.tv.com/news/falling-skies-season-3-premiere-review-cowboys-and-aliens-137061338448/) waiting for me over on the media center. Needless to say, I wanted something I could set and forget for a few minutes.

I’ll use System Image Utility, I thought. I set about creating a NetBoot set using System Image Utility, and used Rich’s method of setting up a VMWare Fusion virtual machine to boot to the .nbi and get things installed. You can do this at home yourself as well. You’ll need the following to get started….

## Requirements:
- OS X Mountain Lion Server (or other recent version) –for the NetInstall service and System Image Utility
- OS X Mountain Lion installer application (downloaded from the Mac App Store)
- VMWare Fusion 5
- Some great shows to catch up on…

## Get Started:
1. Install and configure OS X Mountain Lion Server.
2. Setup the NetInstall service. A good walkthrough of Steps 2 and 3 is available here.
3. Use System Image Utility to build the NetBoot set (.nbi). See italics at Step 2.
4. Use [Rich’s procedure](https://derflounder.wordpress.com/2013/01/23/building-mac-test-environments-with-vmware-fusion-netboot-and-deploystudio/) for setting up a virtual machine to NetBoot using your .nbi. (See ‘Configuring the VM’ section of his post.)

## Notes:
- In my setup, I was able to netboot the virtual machine to the netboot set while running both on the same system.
- The VM should begin with the OS installation prompts at such time that the NetBooting process has completed. Follow the prompts to install and reboot.

![screenshot](/img/mavericks_vm_siu.png)
