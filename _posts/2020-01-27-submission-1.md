---
layout: post
title: "Submission 1"
date: 2020-01-27
---
## Recall
In this first week of the Summer studio labs I've learnt to:
1. solder components onto a Printed Circuit Board (PCB) and send sensor data from the resulting IoT Data Logger (IDL) to the Blynk app on my phone;
2. connect a soil moisture sensor and a 4-digit 7-segment display to an Arduino Uno and program the latter to display the data from the former using the Arduino IDE;
3. connect the soil moisture sensor to an ESP32 and use the PlatformIO extension on VSCode to program it, then send the data to the Blynk app on my phone;
4. set up a portfolio on Github Pages using Jekyll and start learning to use flexbox and SASS/SCSS to style it.

## Reflect
1. My first impression was that solder paste is *awesome* - the 'baking' process was fascinating and the end result was really neat! It was my first time ever doing any soldering though, so the manual soldering made me a bit nervous. Soldering lessons:
  - go left to right to avoid joins (since I'm right-handed);
  - should learn to identify good/bad soldering results;
  - feed the solder a bit.  
  Overall it was a fun process and the fact that the IDL just works is great! The PCB coupled with Blynk really made it quick and easy to get set up and running.  

2. For my prototype I'm planning to make something I can use to help me grow succulents. The first sensor that caught my eye was a capacitive soil moisture sensor. Initially I was only concerned with getting the sensor connected, but soon I realised I needed a display for the data. At first I did not know what components I had in the box, so I didn't realise that I actually had an LED display I could've used. I just went for the 4-digit 7-segment display because that component was an obvious one.  
  ![Open notebook with 4-digit 7-segment display on top of it displaying numbers](/assets/images/week01b.JPG)
It was fun hooking up the display anyway, and it provided some hands-on experience with the Arduino IDE. The only issue I ran into was that the ones digit refused to display when the sensor values had more than one digit. Before I started troublehooting it, however, I had a chat with Mahasvin who turned out to be doing a similar prototype (except that his seemed more ambitious than mine). He pointed out that I should be trying to get the data onto Blynk instead and that I needed a board with Wifi, which made a lot of sense.  

3. This was basically me starting over. Danon walked me through the setup so I was able to get things up and running on Blynk. Getting started is really the hardest thing, especially when I didn't even know what the name of the board was - if I'd known I could've started by looking it up. I now know that the name is actually printed on the board itself though.
  ![Several components connected to breadboard and circuit board with many jumper wires](/assets/images/week01c.JPG)

4. I've been curious about Github Pages for a bit now - just didn't expect that I'd end up using it for class purposes. I was pleasantly surprised to discover that the setup process was actually quite painless and took only about a couple of hours following [this tutorial](http://jmcglone.com/guides/github-pages/) (disclaimer: I was already familiar with Git and also already had a Github account). I then realised that I didn't actually have any experience with styling and therefore had no idea where to even begin, so my weekend was spent working on this, and I found a couple of useful web dev resources ([Colormind](http://colormind.io/bootstrap/) and [Google Fonts](https://fonts.google.com/)). The result was still unsatisfactory to me, however; I've since decided to make this one of my learning objectives.  

Looking back I realise I've actually done **a lot** this week! Things went by quite quickly so I can only hope I've absorbed what I needed. I appreciate the way this subject lets students take control of their own learning. I am not used to setting my own learning objectives, so I personally found it difficult to do, but at the same time being able to direct my own learning in terms of what I want to learn and how, feels...liberating.

## Next Steps
- Add a light sensor to the existing prototype
- Look for examples of web design to emulate