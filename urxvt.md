---
layout: page
title: Urxvt
permalink: /urxvt/
---

![urxvt]({{ site.baseurl }}/images/urxvt.png)

    *foreground:        #B2A191
    *background:        #171717
    *borderColor:       #171717
    *cursorColor:       #C7A551
    !black
    *color0:            #202020
    *color8:            #404040
    !red
    *color1:            #BF3F34
    *color9:            #FF6C5F
    !green
    *color2:            #707D22
    *color10:           #B8CA4B
    !yellow
    *color3:            #BF7A29
    *color11:           #C7A551
    !blue
    *color4:            #627A92
    *color12:           #95B9DE
    !magenta
    *color5:            #75507B
    *color13:           #AD7FA8
    !cyan
    *color6:            #757978
    *color14:           #9FA590
    !white
    *color7:            #B2A191
    *color15:           #E8D0C3
    
    !enabling clickable links:
    URxvt.urlLauncher:      /usr/bin/google-chrome
    URxvt.matcher.button:   1 
    URxvt.colorUL: #627A92
    
    !urxvt scrolling options and cursor style:
    
    urxvt*saveLines: 120000
    urxvt*scrollstyle:plain
    urxvt*scrollBar: false
    
    urxvt.transparent: true
    urxvt*shading: 10
    urxvt*font: xft:terminus:pixelsize=14:antialias=false
    urxvt*boldFont: xft:terminus:bold:pixelsize=14:antialias=false
    urxvt*cursorBlink:      true
    
    Xft*dpi:                96
    Xft*antialias:          false
    Xft*hinting:            full
    
    urxvt.perl-ext-common:  default,matcher

