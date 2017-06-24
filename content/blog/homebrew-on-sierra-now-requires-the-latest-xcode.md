+++
date = "2016-11-12T11:15:06-07:00"
draft = false
title = "Homebrew on Sierra now requires the latest Xcode"
tags = ["Apple", "CI", "Homebrew", "Xcode", "Software", "Sierra"]
categories = ["Development", "Apple", "macOS", "iOS", "CI"]

+++

While fumbling my way through a bootstrapping workflow for macOS Sierra, I found that recent updates to Homebrew [now require the latest version of Xcode](http://discourse.brew.sh/t/xcode-8-0-is-outdated/346) when the machine in question is running macOS Sierra.

It looks like [the check](https://github.com/Homebrew/brew/blob/564fa8867dd865c686d243ba48c202e0cb6a35fe/Library/Homebrew/extend/os/mac/diagnostic.rb#L17) boils down to a scenario when the relevant machine is running macOS Sierra AND does NOT have the CI attribute.

![homebrew-check](/img/homebrew-check.png)

If the CI attribute is specified, then the Homebrew check will proceed and allow a different version of Xcode to be installed. I imagine the Homebrew maintainers will continue to monitor the latest versions of Xcode and update the check to look for the most recent version. For reference, you can check out the discussions around this topic [here](https://github.com/Homebrew/brew/commit/12aad5c65fee39c5f044e39ca1efcbed58aebd39#comments) and [here](https://github.com/Homebrew/brew/issues/972) on the thinking around this change.

In a future post, I'll talk about how to bootstrap a machine using tooling like Puppet, and how to expose this environment variable to the user environment and to Puppet so that workflows to bootstrap machines proceed without issue.
