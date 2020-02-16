---
layout: post
title: "Submission 3"
date: 2020-02-16
---

### Sections

- [Sprint 3: Connecting the IoT hardware prototype to the cloud infrastructure](#sprint-3-connecting-the-iot-hardware-prototype-to-the-cloud-infrastructure)
- [Sprint 4: Freestyle](#sprint-4-freestyle)
- [Artefact #1: Soil moisture sensor data visualisation](#artefact-1-soil-moisture-sensor-data-visualisation) + [Reflection](#reflection-on-artefact-1)
- [Artefact #2: Light sensor data visualisation](#artefact-2-light-sensor-data-visualisation) + [Reflection](#reflection-on-artefact-2)
- [Artefact #3: Temperature sensor data visualisation](#artefact-3-temperature-sensor-data-visualisation) + [Reflection](#reflection-on-artefact-3)
- [Artefact #4: Setting up soil moisture alerts](#artefact-4-setting-up-soil-moisture-alerts) + [Reflection](#reflection-on-artefact-4)

## Sprint 3: Connecting the IoT hardware prototype to the cloud infrastructure

This sprint I spent quite a bit of time trying to help a couple of my team mates get their own cloud infrastructure set up. They wanted to be able to set it up on their own, so they followed the same tutorial I used. For some reason they seemed to run into more issues than I did though; they got error messages that I didn't get, so I found it challenging to try to help them resolve those issues. It didn't help that, like me, they didn't have Linux and Docker experience either. In the end, I was only able to successfully help one of them. It then turned out that we *had* to share one database anyway, so we're using the one I set up as it is already running. I don't think they would have any issues connecting to it, instead the main problems would likely have to do with the format of their MQTT messages as it has to follow the InfluxDB line protocol syntax that seems to be quite pedantic. I hope they've checked out the link I posted in our Teams channel.

Tasks:
- [x] Connect soil moisture sensor and visualise its data
- [x] Connect light sensor and visualise its data
- [x] Connect temperature sensor and visualise its data
- [x] Map raw soil moisture readings to percentage
- [x] Install and run Chronograf on the server
{: style='list-style-type: none'}

## Sprint 4: Freestyle

This sprint we were free to choose what we wanted to work on, so I chose to focus on completing my prototype (as much as I could anyway). We were told that in the next sprint we'd be shrinking it down and starting on mechanical design for an enclosure to house it. I thought I'd better finalise my prototype. I had last minute ideas that I could not get to this sprint, however, such as:

- Adding a grow light to my device that I could potentially switch on and off in response to detected light levels; and
- Setting up Over The Air (OTA) firmware updates for my prototype (Danon introduced this idea to me).

Tasks:
- [x] Send alerts to mobile phone
- [x] Speed up data pipeline: configure Telegraf data flush interval from 10 seconds to 1 second (`flush_interval` setting in `agent` section of the configuration file)
- [ ] ~~Try the *IoT OnOff* iOS app for data visualisation on the mobile phone~~ (Grafana seems to work fine in the mobile browser)
{: style='list-style-type: none'}

## Artefact #1: Soil moisture sensor data visualisation

At this stage, my hardware prototype was still bare-bones and consisted of only a capacitive soil moisture sensor. For this milestone artefact, the goal was just to connect the sensor to the infrastructure and get the data to be visualised in Grafana; i.e. have the device connect to the MQTT broker (Mosquitto) that I had set up on my server and send over the sensor readings.

### Planning

According to [the documentation](https://docs.influxdata.com/influxdb/v1.7/introduction/getting-started/#writing-and-exploring-data), the InfluxDB line protocol (`INSERT` statement syntax) consists of:
- time(stamp) - optional; automatically added
- measurement - conceptually equivalent to the SQL table
- field e.g. `value=0.62,temperature=27.8` - must have at least one; non-indexed column
- tags - metadata; can have zero to many; indexed column

Considering that the team might have to share the one database, I thought I'd use my name as the measurement (table name). I would have `pot=` as a tag (metadata) to identify individual pots, and the field would be `soil_moisture=`. The MQTT message to be published, that would effectively be my database `INSERT` statement, would therefore be: `catt,pot=1 soil_moisture=`.

### Execution

I tried combining the relevant bits of my last working code from Sprint 1 with Daniel's code that worked in Sprint 2. There were many lines that I assumed were specific to his sensors so I looked up the [original code template](https://platformio.org/lib/show/5956/EspMQTTClient) that he used, and experimented with it. I tried outputting to the Serial Monitor first in order to check the sensor readings, but it was showing gibberish. Looking it up, this turns out to be an issue with the monitor speed, and it was resolved by adding the line `monitor_speed = 115200` to the project configuration file.  

The device was able to connect to Mosquitto successfully, but the sensor readings seemed to be stuck at the value 4095 whether it was dry or dipped in water... I remembered then that I'd dropped it in class, and wondered whether I'd broken it :( I decided to try running my Sprint 1 code that connected to the Blynk app on my phone and connected the sensor to a graph visualisation so I could see whether it would change over time, but it didn't work either...until I realised that I was using a different pin for that project: analog pin 39, compared to analog pin 2 for the current project, and when I connected the sensor to 39 instead, it started working like it did in Sprint 1 (yay!).  

All that's left for me to do was to change the pin I was reading from in my current project, print the sensor readings to the Serial Monitor to confirm that it's working as intended, then make small changes to the code to publish the data to Mosquitto, finally set up a graph for it in Grafana, and...voila!

![Graph of soil moisture sensor readings in Grafana](/assets/images/week-03a.jpg){:.original-size}

## Reflection on Artefact #1

My code seems to be a lot simpler than Daniel's, which I'm glad about. I wonder if it's got to do with us using different sensors. I guess I'll find out as I add more sensors to the mix. I'd like to add a light sensor next.  

While setting up the graph on Grafana I realised that I might eventually need to tweak my MQTT message (database `INSERT` statement). I don't think I've thought out the data structure well enough just yet. I haven't fully considered what my data needs might be down the track and the different ways I might want to query the data. I can think of a couple of ways to structure it:

1. `catt,pot=1,measure=soil_moisture value=`
2. `catt,pot=1 soil_moisture=`

I've gone with option 2 at the moment, as this way I can add more sensors to the one message.

## Artefact #2: Light sensor data visualisation

After Artefact #1 was successfully set up, my next step was to add a light sensor to the mix, and hopefully be able to pipe the readings through to Grafana like I did with the soil moisture data.

### Initial research

I first looked up the light sensor that I got from Danon, and it appears to be the [Adafruit GA1A12S202 Log-scale Analog Light Sensor](https://www.adafruit.com/product/1384). Conveniently there is also a [getting started guide](https://cdn-learn.adafruit.com/downloads/pdf/adafruit-ga1a12s202-log-scale-analog-light-sensor.pdf) for it that shows how to wire and program it with a sample sketch. This is geared towards Arduino boards but I thought I'd just give it a try on my ESP32 board and see how it goes.

### Planning

For the addition of the light sensor, I wanted to send the data all in one go; each MQTT message published would contain all the data for the pot. To achieve this, I added a `light=` field to the existing message format, e.g. `catt,pot=1 soil_moisture=25,light=100`.

### Connecting the light sensor

Following the guide linked above, the wiring seemed quite straightforward. My main hurdle at this point was my lack of understanding of the hardware: I did not really know what difference it would make whether I connect the VCC to the 5v pin or the 3.3v pin. When I read up on it apparently it affects the accuracy of the sensor. The guide instructs to connect the VCC to the 5v pin, then 3.3v to the AREF on the Arduino board, but I am working with an ESP32 which does not have any AREF pin. I ended up trying the VCC on both the 5v and 3.3v pins, and it seemed there was a difference in the readings. Measuring the brightness of my phone flashlight, at 3.3v I was getting readings between 1000 to 2000 lux, while at 5v the readings were mostly between 3000 to 4000 lux.

### Coding for the light sensor

I started a new PlatformIO project in order to have a clean slate from which I could try out the sample sketch in the getting started guide. It was short and straightforward, so once I confirmed the sensor was connected and functional, I added the code to my prototype codebase. It was starting to become unwieldy to be building the MQTT message by appending strings with `+` however; I wondered whether there was a string builder function I could use instead. Then, while trying to help Malan with her code I happened to see that she was using `snprintf()`. This turned out to be a buffer which I then used to build my message before publishing it. I think it makes my code more readable.

Danon also advised me to use the `BlynkTimer` library instead of the `delay()` I'd been using. Luckily for me I had used it in the Sprint 1 code that he walked me through, so it was relatively easy to replicate the usage in my current codebase. To be honest I didn't think I noticed any difference, but it turned out `delay()` basically paused the processor and blocked other processes - it did not allow multi-tasking (reference: [Ditch the delay()](https://learn.adafruit.com/multi-tasking-the-arduino-part-1/ditch-the-delay)).

Adding the light sensor was a relatively smooth and easy process thanks to the guide. All that was left was to set up a new visualisation panel on the dashboard.

![Serial Monitor output in the Visual Studio Code terminal panel](/assets/images/week-03h.jpg){:.original-size}

![Dashboard with soil moisture data and light data graph panels](/assets/images/week-03b.jpg){:.original-size}

## Reflection on Artefact #2

I was a bit torn between feeling like I should slow down and try to have a deeper understanding of the pins, and the impatience of wanting to just make it work (somehow) and move on to my next tasks. The former felt like it could potentially be a rabbit hole though; I ended up going with the 3.3v pin which in hindsight seemed to be the one compatible with the next sensor anyway.

So far I'm finding the process relatively smooth-sailing as long as I could figure out what the component was and it had a beginner-friendly guide I could follow. As long as there were examples I could learn from and replicate/apply, it seemed to be all good. The cloud infrastructure setup last sprint was like that too. If only everything was like this :)

## Artefact #3: Temperature sensor data visualisation

Now that I had soil moisture and light data, I thought the next (and possibly last) parameter I needed was temperature. Succulents tend to be able to tolerate heat better than cold conditions, so I thought it'd be good to be able to collect temperature data and monitor its effect on the plants.

### Research and preparation

Initially, I thought that the sensor that I got was a [DS18B20 waterproof temperature sensor probe](https://www.banggood.com/2-Wires-Cables-Digital-Temperature-Sensor-Probe-DS18B20-Thermometer-Thermal-p-986819.html?ID=44707&cur_warehouse=CN), but it turned out to be a [NTC thermistor](https://www.e-tinkers.com/2019/10/using-a-thermistor-with-arduino-and-unexpected-esp32-adc-non-linearity/) instead. They looked similar! I couldn't have figured this out without Danon's help. He helped me connect and test it too. We eventually decided that it wasn't behaving as we expected, however; since I wanted to measure environmental temperature instead of soil temperature, he gave me a different sensor to use: a [BME280](https://learn.adafruit.com/adafruit-bme280-humidity-barometric-pressure-temperature-sensor-breakout/arduino-test). Aside from temperature, this sensor also measures humidity, pressure and (estimated) altitude. I soldered it myself under Danon's guidance :)

![Me soldering the BME280](/assets/images/week-03c.png)

### Connecting the temperature sensor

The soil moisture sensor and light sensor each has 3 pin connections, and I *think* I can handle that configuration now. This temperature sensor is more complicated though - it has 6. Danon walked me through the wiring, starting with an explanation of 2 bus communication protocols that we could use to set the sensor up: I2C and SPI. The latter looked quite complicated and seemed to require quite a few wires; we went with the former.

![Hand-drawn diagrams of the I2C and SPI bus communication protocols](/assets/images/week-03e.JPG)

Luckily, even though there were more pin connections to make compared to the other sensors, they were mostly made obvious by the labels on the sensor and on the development board. The only tricky bit was that the default protocol configuration was SPI, so we had to change it to I2C by wiring the SDO to VCC.

![BME280 temperature sensor](/assets/images/week-03g.JPG)

![Hand-drawn diagram of the extra wiring needed to configure the sensor to use I2C protocol instead of SPI](/assets/images/week-03f.JPG)

### Coding for the temperature sensor

We looked up example code for the BME280 on Platform IO > Libraries. One thing to note was that the protocol required the sensor to be assigned an address (`0x77` in this case). Aside from that, the usage was as simple as calling `.readTemperature()`, `.readHumidity()` and so on. I ended up not using the sensor's pressure and altitude outputs as they didn't seem relevant to my prototype.

By this point, I was just appending the `temperature=` and `humidity=` fields to the MQTT message, e.g. `catt,pot=1 soil_moisture=25,light=100,temperature=22,humidity=33`. All that was left was to set up the corresponding graphs/visualisations in Grafana.

![My Grafana dashboard with multiple types of visualisations](/assets/images/week-03d.png){:.original-size}

## Reflection on Artefact #3

Learning about the bus communication protocols reminded me of a networking class, except that instead of connecting computers to a switch/router we were connecting sensors to a development board. I remembered an article I came across while researching for a networking assignment - if I remembered correctly it was about the design for a huge network of sensors in the context of underground mines. Given how miniscule my prototype was, I thought it was good to have this learning opportunity I might otherwise not have.

While looking for example code, I realised that I usually defaulted to searching Google, even when I had the Platform IO library available to me. It felt a bit silly because there was no need to search the whole Internet when the latter would already contain the most relevant examples. Duh.

## Artefact #4: Setting up soil moisture alerts

What I had in mind was a prototype that monitors my succulents, visualises the sensor data on a dashboard and also send me alerts when the sensor readings reach certain thresholds, e.g. when the soil is dry I should get an alert to tell me it is time to water. I think the only feature that my prototype is missing at the moment is the alert capability.

I had come across Kapacitor mentioned multiple times while looking up references for setting up the cloud infrastructure in Sprint 2. It is the final component of InfluxData's TICK stack: Telegraf, InfluxDB, Chronograf and Kapacitor. Kapacitor is a real-time data processing engine with the ability to send alerts based on a specified data trigger. I installed and set it up to run in a Docker container on the server.

Kapacitor has many alert handlers to choose from, including email, MQTT, HTTP POST and Slack (see [this list of supported handlers](https://docs.influxdata.com/kapacitor/v1.5/working/alerts/#list-of-handlers)). I wanted to receive the alerts on my phone, so I decided to go with the Slack alert handler. As for the other possiblities I considered:
- Email: To be honest I didn't want to get more emails than I already do (and I would've had to set up a SMTP server)
- MQTT: Could possibly work with the IoT OnOff iOS app, but I prefer and am already familiar with Slack
- HTTP POST: I would've had to set up an endpoint for it... :/

Aside from the connection to InfluxDB, the changes to be made to the Kapacitor configuration file depended on which alert handler(s) were being used. The [official documentation for Slack alert handlers](https://docs.influxdata.com/kapacitor/v1.5/event_handlers/slack) provided the necessary details. In my case, I set up a new Slack workspace for my personal use, added a #succulent-monitor channel for the alerts, and created an incoming webhook in Slack for Kapacitor.

I then followed [the official guide](https://docs.influxdata.com/kapacitor/v1.5/introduction/getting-started/#triggering-alerts-from-stream-data) to setting up the alerts. I needed to set up a script to let Kapacitor know which data to pay attention to, what the trigger condition was, and what alert message to send to which alert handler. It turned out alerts can be set up in Kapacitor in 2 ways:

1. Using TICKscript only (which I did; see below)
2. Using TICKscript that publishes to a topic, and handler(s) (defined in `.yaml` files) that subscribe to the topic and send the alert messages

The second method seems to allow more customisation, such as aggregating alerts over a specified time period or filtering them by level (`OK`, `INFO`, `WARNING` and `CRITICAL`). I plan to try this out, but in the meantime this was what the first version of my script looked like:

```
stream
  |from()
    .measurement('catt')
  |alert()
    .warn(lambda: "soil_moisture" < 5)
    .stateChangesOnly()
    .message('Pot is dry. Time to water')
    .slack()
      .workspace('ctantri')
      .iconEmoji(':cactus:')
```

I've since had to increase the threshold from 5 to 20 because the sensor readings fluctuated quite a bit and I ended up with waaay too many alerts during testing :(

![Screenshot of alerts received on Slack](/assets/images/week-03i.PNG)

**Note 1:** There was some troubleshooting involved during testing - Kapacitor was not able to connect to InfluxDB because it turned out I missed one line in the configuration file that specified the Kapacitor IP address. Oopsie :( I restarted Kapacitor and the database a few times while troubleshooting to see whether the new IP address would flow through on its own, but in the end I had to [manually create a new subscription in InfluxDB](https://docs.influxdata.com/influxdb/v1.7/administration/subscription-management/#create-subscriptions) before it would send data to Kapacitor.

**Note 2:** I noticed that there was an alert generated every time the threshold is crossed *in both directions*, i.e. both when the soil moisture level goes *up* above 20% and when it goes *down* below 20%. I only wanted to be alerted of the latter :(

**Note 3:** According to [the documentation](https://docs.influxdata.com/kapacitor/v1.5/tick/introduction/#stream-or-batch), Kapacitor can process data either in real-time (as a stream) or in batches. In the case of a stream, it is possible to process and transform data before it is written to InfluxDB.

## Reflection on Artefact #4

Firstly, it seemed that I was getting comfortable with adding to the cloud infrastructure, perhaps because Kapacitor was also an InfluxData product - the setup procedure was similar to that of the rest of the TICK stack. I set it up without following any step-by-step tutorial this time around. I just referred to the [documentation for the Docker image for Kapacitor](https://hub.docker.com/_/kapacitor) for the `docker pull` and `docker run` commands, along with the command for generating a default configuration file.

Secondly, I felt I got plenty of practice using Docker commands while testing and troubleshooting, mainly to do with running, stopping and restarting a container, and also executing commands inside specific containers. I also learnt how to copy files into a container. All in all it was a really productive learning experience that ended up making me feel *a bit* more confident using Docker :)

Lastly, after running the tests and seeing the number of alerts generated within a minute I was rather relieved that I had chosen Slack over email... I realise that I do not actually need to know *immediately* when the soil becomes dry; I just wanted *one* alert to let me know within a day of it happening, so I plan to try calibrating the alerts by batching/aggregating/filtering.