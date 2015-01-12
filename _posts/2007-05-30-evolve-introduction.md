---
layout: post
title: Evolve - Introduction
date: '2007-05-30T08:14:00-04:00'
tags:
- Enlightenment
- enightenment
- evolve
- edje
tumblr_url: http://www.hisham.cc/post/30203502818/evolve-introduction
---
Over the past couple of weeks, I have been working on Evolve. Evolve started out as a parser that can parse something like:

    widget
    {
      type: "window";
      name: "main_window";
      requsted-width: 320;
      requested-height: 240;
      title: Ëvolve Main Window";
    }

and convert that into Etk code. The main idea was for you to be able to design the interface of your Etk app using some Edje like kanguage so you would not need to mess with all that C code to create the GUI (painful).


After that functionality was achieved, I wanted Evolve to do more. So I added signals, callbacks, and the ability to pass widgets to those callbacks using Evolve itself. With that, Evolve would create a complete representation of your GUI in memory and it could use that to construct it. The next step, naturally, was to allow Evolve to take that data structure and write it to a binary file (using Eet). Later on came image integration and storage into the binary itself. The result of this was that you could create a GUI, embed your images, and give it to your application developer who would concentrate on the applications logic rather than waste time working out the details of the GUI. Several GUI’s can be created for the same application.



This was starting to look good, but it was not good enough. We needed the ability to theme the Etk widgets in the Evolve binary, and we needed the ability for the app using Evolve to look and feel like any Edje application would, removing the restriction to look and feel simply like your everyday Etk application. This was the next step. After finishing this step, any Evolve (and eventually Etk) widget in your application could set its own appearence using Evolve’s Edje integration. Evolve allows you to set a custom Edje file and group for your widget, and it allows you to have your Etk widgets embedded in an actual Edje application. The result would be like the modular9 screenshot included here.



So now we had a way to design the GUI using an Edje like language, embed Etk widgets in an Edje application, and custom theme all our widgets, wonderful. What we needed was for the Edje and Evolve binaries to be a single file, a single look and feel file, so that was the next step. Having done that, the next thing I wanted to work on was an Evolve builder application that allowed the user to build his GUI using a very simple visual interface by selected widgets and placing them in windows. The Evolve builder is shown in the other screenshot and here.



The final, and yet uncompleted steps are the ability for using Edje to set the look and feel of the Evolve GUI form the builder itself, and including live previews for everything, and eventually allowing the user to create and edit Edje interfaces (using our Edje editor) from the Evolve builder itself.
