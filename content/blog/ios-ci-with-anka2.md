---
title: "iOS CI with Anka Virtualization - Part 2"
date: 2017-07-16T14:13:58-07:00
draft: true

---

_Disclaimer: I am an advisor for Veertu Inc._

## Introduction 
I am writing a series of articles to show you how to get started building your own macOS private cloud for iOS CI using [Veertu Inc.'s](https://veertu.com) latest technology, [Anka](https://veertu.com/anka-technology/). Anka was built to support many of the principles foundational to good continuous integration systems, including supporting fast incremental deploys and immutable infrastructure concepts. In this series, I'll show you how to get started with their hypervisor, registry, and controller to take full advantage of tools that were purpose-built to support iOS and macOS CI on Apple hardware.

In [Part 1]() we installed [Anka Build]() and created our first VM. In this post (Part 2) we'll talk about the concepts of immutable infrastructure and what workflows you can use with Anka to enable this.

## Immutable Infrastructure
[Immutable Infrastructure](https://www.oreilly.com/ideas/an-introduction-to-immutable-infrastructure) is a systems engineering concept derived from programming principles where infrastructure configuration is unchanged once deployed. While the concept may vary amongst deployments, the overall idea supports configuration of a base, and any instances of that infrastructure are provisioned from said base. If an instance encounters an issue, the instance can be replaced easily with an updated configuration if required.

**Anka was designed to support immutable infrastructure concepts.** Each Anka VM you create can be loaded with any configurations, modified user or system preferences, and build or test dependences. Once that VM is provisioned, it can be versioned and pushed to [Anka Registry]() for storage and distribution throughout environment for repeated use. If an update to your Anka VM is required (Xcode, ruby gem, etc.), you simply make the configuration change in the base, bump the version, and push to Anka Registry again for usage throughout your environment. 

## In Practice
Since we created the `macos-anka-v1` VM in [Part 1](), let's use this VM to see how Anka enables immutable infra workflows. Our base was easy to create, but we didn't make any changes to the base operating system, so let's do that now. 
