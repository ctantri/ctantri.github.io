---
layout: post
title: "Submission 4"
date: 2020-03-03
---

![Laser-cut acrylic pieces next to prototype components](/assets/images/week-04o.JPG)

### Sections

- [Sprint 5: Mechanical design](#sprint-5-mechanical-design)
- [Artefact #1: Incorporating an ADC](#artefact-1-incorporating-an-adc) + [Reflection](#reflection-on-artefact-1)
- [Artefact #2: Wiring and soldering](#artefact-2-wiring-and-soldering) + [Reflection](#reflection-on-artefact-2)
- [Conclusion](#conclusion)
- [Final reflective statement](#final-reflective-statement)

## Sprint 5: Mechanical design

The focus of this sprint was the design of the enclosures to house our prototypes. At the very start of the sprint, we learnt the basics of Fusion 360, a 3D design software, by following along while Danon gave us a demonstration building an ESP32 model and an enclosure for it.

I had struggled quite a bit this sprint mainly because it took me a long time to conceptualise the final form I wanted my prototype to take, and also because the whole workflow was so unfamiliar to me. I was happy with how the end product turned out, however. It was a complete, functional product :)

![Completed prototype](/assets/images/week-04p.JPG)

Tasks:
- [x] Add ADC to prototype
- [x] Solder components
- [x] Design enclosure(s)
- [x] Crimp and add header for I2C cable
{: style='list-style-type: none'}

## Artefact #1: Incorporating an ADC

While I was struggling with my end product concept, Danon helped me simplify it by breaking it down into two separate units: the sensors make up one unit while the rest makes up the other unit. This is because the 'rest' includes a battery, which should not be left out in the sun with the sensors unit. The two units would then be connected with a cable using the I2C protocol.

![Diagram on whiteboard](/assets/images/week-04a.JPG)

The tricky thing about this new arrangement was that I needed to incorporate an analog-to-digital converter (ADC). My understanding was that in order to for all my sensors to communicate through that one cable they would all need to be transmitting digital signals, which my temperature/humidity sensor already did, but my soil moisture sensor and light sensor were both analog. Enter the ADC.

![Analog to digital converter](/assets/images/week-04b.JPG)

Michaela was already using the same ADC for her heartrate sensor, so she generously guided me through soldering headers onto my ADC and gave me a walkthrough of the code for her ADC setup. We couldn't get it to work on mine, however. It was more complicated than I expected, and I ended up finding and using another library - the [Adafruit ADS1X15](https://microcontrollerslab.com/ads1115-external-adc-with-esp32/) - that was simpler to implement due to its very minimal setup.

![Rough prototype of sensors on breadboard with jumper wires](/assets/images/week-04c.JPG)

At this point, the setup was somewhat working. I found the soil moisture sensor readings more stable than before, but the light sensor readings didn't seem accurate. As shown in the photo above though, it was still far away from a practical prototype.

## Reflection on Artefact #1

It felt like I was taking a shortcut by using the Adafruit ADS1X15 library, at the expense of greater control over the setup, but at the same time knowing that it would take me a much longer time to work out the setup details otherwise because I do not have the electrical background. At this point I thought I'd rather prioritise getting it to work as quickly as I could because there was still a lot to do.

## Artefact #2: Wiring and soldering

![Prototype boards with mini ESP32, battery cell and battery charger module](/assets/images/week-04d.JPG)

The next step, which to me was a major one, was to shrink down the prototype. I was given an alternative ESP32 board in a smaller form factor, and the main idea was to make a smaller, more permanent prototype by replacing the breadboard with small prototype boards that we'd solder the components to. This took me a long time because it required me to first decide what my end product should look like. I wanted my sensor unit to be positionable using the soil moisture sensor as a support pillar so it could be stuck into the soil in a succulent pot while holding up the other sensors.

I also had to position the components on the prototype boards and plan the wiring. This was an unfamiliar workflow to me. It seemed that others were able to solder theirs together by referring to a circuit diagram, while I needed a more specific plan. Luckily for me the holes in the prototype board were labelled, in a similar way to an Excel spreadsheet, so I could plan the wiring in detail using a drawing app on my phone (it features layers so I was able to place each colour in its own layer so they do not interfere with one another). I also found that I did not need the whole length of the prototype boards, so Danon ended up cutting them for me (I wasn't allowed to operate the saw).

![Planned positioning of sensors on prototype board](/assets/images/week-04e.JPG)

![Planning sheet with stencilled prototype board](/assets/images/week-04f.JPG)

![Wiring plan in a drawing app](/assets/images/week-04g.PNG)

![Prototype board being cut using a scroll saw](/assets/images/week-04h.JPG)

It then took me practically a whole day to carry out the soldering. In hindsight that was because I tried to cut the wire lengths as close as possible and positioned them as neatly as possible, despite the pins being really close to one another and my lack of experience. Michaela generously let me use some of the pretty pastel wire in her stash :)

![Soldering work in progress](/assets/images/week-04q.JPG)

I did the same for the other unit, which consisted of the mini ESP32, a Nokia battery, a battery charger module, a power jack and a switch. Michaela kindly explained to me how the components were to be connected by drawing up a circuit diagram. This unit was a bit more complicated because the power jack and switch had to be positioned off the prototype board, while the battery had to be added last because it needed a case/holder (fortunately Nick had designed one so I just needed to have it 3D-printed).

![Circuit diagram](/assets/images/week-04i.JPG)

![Planning sheet with stencilled prototype board](/assets/images/week-04j.JPG)

![Wiring plan in a drawing app](/assets/images/week-04k.PNG)

![Wiring on the back of the prototype board](/assets/images/week-04m.JPG)

![Components on the front of the prototype board](/assets/images/week-04n.JPG)

## Reflection on Artefact #2

This artefact was only completed very close to the end of the session; to be honest I nearly panicked, thinking I might not be able to finish my prototype. I felt that I had gained some confidence with soldering, and under more relaxed circumstances, would have enjoyed it more :)

## Conclusion

I had only a very vague idea of what I wanted to do for my prototype in the beginning, and my approach to it had mostly been exploratory, so I'd had to modify my learning contract to include more details as they became clearer to me along the way.

I was quite excited at the prospect of building my own blog and saw it as an opportunity to also learn more about web design, so I included SASS/SCSS and Flexbox in my learning contract. While I did manage to learn a bit about them at the beginning and incorporate a bit of them in the blog, I had unfortunately not made progress since. It seemed that I had underestimated the workload for the rest of my learning contract. However, I believe I have managed to prepare the deliverables and meet my own assessment criteria (albeit barely in the case of the website styling ones).

## Final reflective statement

I think that choosing a project that addressed my own problem was very beneficial for my learning. Having a personal stake in it meant that I had a better idea of what features were required and was also more invested in getting it done. Looking at the completed end product, I'm proud of how far I've come in six weeks.

Having no electrical background had meant that I struggled with trying to understand how things work. I could try to understand basic circuits, but somehow electrical concepts went over my head. Fortunately I have not had to dwell too long at this level; Danon and my classmates (especially Michaela) had helped me a lot to overcome those blockers. I would have needed a lot longer than six weeks to figure things out on my own. Also, where we shared common components in our prototypes, my classmates shared their 3D models of those components with me so that we could save time.

As an IT student starting from practically zero in electronics, I was more confident with building the cloud infrastructure because that was a relatively more familiar ground. The software side of things was easier because of the libraries and examples available out there. They saved me time by allowing me to work at a higher, abstract level instead of having to interface directly with the hardware, which I would have struggled with.

Having said that, I also enjoyed the hands-on work with the hardware. It felt a bit more satisfying compared to my past software projects because it resulted in a tangible product that I could use in my daily life. It also enriched my learning experience and made it fun by allowing me to traverse the layers of technology, from abstract software to circuits and back.