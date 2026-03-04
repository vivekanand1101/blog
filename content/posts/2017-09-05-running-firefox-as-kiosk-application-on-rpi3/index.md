---
title: "Running Firefox as kiosk application on RPi3"
date: 2017-09-05
categories: 
  - "tech"
---

At my day job, i had to run firefox as a kiosk application on RPi3. In this blog post, i will note down the steps that i did so that i or my team members can refer to it when needed.

I have not used any display manager or Desktop Environment but had to use matchbox-window-manager to make firefox run on full screen.

1. _sudo apt-get install vim_ (this is just for me, i can't help it)
2. _sudo apt-get install xorg xutils matchbox-window-manager_ 
3. _sudo apt-get install iceweasel_ (this is firefox :p)
4. _sudo raspi-config_
    1. Go to boot options and setup auto login for user pi
    2. Change the keyboard layout if you wish to
5. As a sudo user, do the following steps:
    1. _cp -r /home/pi  /opt/_
    2. _cd /opt/pi_
    3. _chmod -R a+r ._
    4. _touch .xsessionrc_
    5. _chmod a+x .xsessionrc_
6. Open .xsessionrc as a sudo user and put the following lines there:
    1. _xset s off # no screen saver_ _xset -dpms  # disable some power consumption thingy_ _xset s noblank # don't blank the rpi screen_ _matchbox-window-manager &_ _while true; do_   _firefox --url http://127.0.0.1_ _done_
7. Copy _.xsessionrc_ file to _/home/pi/_ 
    1. _cp .xsessionrc /home/pi_
8. Configure _.bash\_profile_ to start X server when user logs in:
    1. _if \[ -z "$DISPLAY" \] && \[ -n "$XDG\_VTNR" \] && \[ "$XDG\_VTNR" -eq 1 \]; then_ _exec startx_ _fi_
9. Install an extension in firefox to apply kiosk mode. The first extension that comes up when you search kiosk in addons works fine

The blog post that helped most in coming up this setup was: [http://www.alandmoore.com/blog/2011/11/05/creating-a-kiosk-with-linux-and-x11-2011-edition/](http://www.alandmoore.com/blog/2011/11/05/creating-a-kiosk-with-linux-and-x11-2011-edition/)
