---
title: "Detecting USB Insertion/Removal using Python"
date: 2018-01-03
categories: 
  - "tech"
---

In my [last blog](https://vivekanandxyz.wordpress.com/2017/12/29/detecting-and-automatically-mounting-pendrive-on-raspbian-stretch-lite/), i wrote about how usbmount can be used to automatically mount a usb. Today, i had to detect this mounting from python and show it in a web application that a usb has been inserted. I used [pyudev](https://github.com/pyudev/pyudev) for this and ran it in a different thread so that the main application thread is not affected by pyudev's monitor loop.

\[code language="python"\] class USBDetector(): ''' Monitor udev for detection of usb '''

def \_\_init\_\_(self): ''' Initiate the object ''' thread = threading.Thread(target=self.\_work) thread.daemon = True thread.start()

def \_work(self): ''' Runs the actual loop to detect the events ''' self.context = pyudev.Context() self.monitor = pyudev.Monitor.from\_netlink(self.context) self.monitor.filter\_by(subsystem='usb') # this is module level logger, can be ignored LOGGER.info("Starting to monitor for usb") self.monitor.start() for device in iter(self.monitor.poll, None): LOGGER.info("Got USB event: %s", device.action) if device.action == 'add': # some function to run on insertion of usb self.on\_created() else: # some function to run on removal of usb self.on\_deleted() \[/code\]

Since, usbmount mounts the filesystem of usb at some particular locations (/media/usb0-7), you can easily check for files in those folders and do whatever you wish to do with it.

Using the above code would be as easy as creating an object of _USBDetector_ class.
