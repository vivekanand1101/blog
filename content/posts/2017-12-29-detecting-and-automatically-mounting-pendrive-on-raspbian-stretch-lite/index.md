---
title: "Detecting and Automatically Mounting Pendrive on Raspbian Stretch Lite"
date: 2017-12-29
categories: 
  - "tech"
---

In the lite version of Raspbian Stretch, i didn't expect that i would have to manually mount the pendrive. I had a use case in which mounting it automatically was a necessity and thus i came across [usbmount.](https://github.com/rbrito/usbmount)

You can install it in raspbian using apt-get.

> sudo apt-get install usbmount

Usbmount gives a lot of configuration options in _/etc/usbmount/usbmount.conf_  but, for my use case the default were good enough.

This wasn't enough though. It wasn't detecting the usb stick. And, on searching i found out that i wasn't the only one who was having [this problem](https://github.com/rbrito/usbmount/issues/2). For the usbmount to work in raspbian stretch, you will have to manually edit systemd file for udevd located in _/lib/systemd/system/systemd-udevd.service_ and change _MountFlags=slave_ to _MountFlags=shared_ as someone commented [here](https://github.com/rbrito/usbmount/issues/2#issuecomment-330739503).
