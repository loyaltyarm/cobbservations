---
title: "iOS CI with Anka Virtualization - Part 2"
date: 2017-07-16T14:13:58-07:00
draft: true

---

_Disclaimer: I am an advisor for Veertu Inc._

## Introduction 
I am writing a series of articles to show you how to get started building your own macOS private cloud for iOS CI using [Veertu Inc.'s](https://veertu.com) latest technology, [Anka](https://veertu.com/anka-technology/). Anka was built to support many of the principles foundational to good continuous integration systems, including supporting fast incremental deploys and immutable infrastructure concepts. In this series, I'll show you how to get started with their hypervisor, registry, and controller to take full advantage of tools that were purpose-built to support iOS and macOS CI on Apple hardware.

In [Part 1](http://cobbservations.com/blog/ios-ci-with-anka-virtualization---part-1/) we installed [Anka Build](https://veertu.com/anka-technology/) and created our first VM. In this post (Part 2) we'll talk about the concepts of immutable infrastructure and what workflows you can use with Anka to enable this.

## Immutable Infrastructure
[Immutable Infrastructure](https://www.oreilly.com/ideas/an-introduction-to-immutable-infrastructure) is a systems engineering concept derived from programming principles. In a most basic sense, infrastructure configuration is unchanged once deployed. While the concept may vary amongst deployments, the overall idea supports configuration of a base and any instances of that infrastructure are provisioned from said base. If an instance encounters an issue, the instance can be replaced easily with an updated configuration if required.

**Anka was designed to support immutable infrastructure concepts.** Each Anka VM you create can be loaded with any configurations, modified user or system preferences, and build or test dependences. Once that VM is provisioned, it can be versioned and pushed to [Anka Registry](https://veertu.com/anka-technology/#registry) for storage and distribution throughout the environment for repeated use. If an update to your Anka VM is required (Xcode, ruby gem, etc.), you simply make the configuration change in the base, bump the version, and push to Anka Registry again for usage throughout your environment. 

## In Practice
Since we created the `macos-anka-v1` VM in [Part 1](http://cobbservations.com/blog/ios-ci-with-anka-virtualization---part-1/), let's use this VM to see how Anka enables immutable infra workflows. Our base was easy to create, but we didn't make any changes to the base operating system, so let's do that now. To help with this, I'm sharing a "bootstrap script" (`bootstrap.sh`) script for provisioning bare metal hardware or VMs for the purposes of macOS/iOS CI. [You can check out the bootstrap script on Github](), and the accompanying [blog post]().

So now that we have the `macos-anka-v1` VM, let's do some configuration. We'll walk through the steps for this manually here, but I **strongly** recommend automating this work in production. We'll do 2 things to get our base fully set up and working for our needs:

1. Run our bootstrap script (installs some dependencies and changes settings)
1. Install Xcode (our biggest dependency ofr iOS project development)

There is also an _optional third step:_ installing a configuration management agent (Puppet, Chef, etc.) and automating additional provisioning steps. As you can see, normally I do this in `bootstrap.sh` to prepare the VM for provisioning. We will install the agent here, but won't use it for any provisioning at this time.

### Bootstrapping our base VM
Add some text about bootstrapping a base VM.

### Provisioning our base VM
Add some text about provisioning a base VM.




