---
layout: post
title: "Submission 3"
date: 2020-02-10
---
![My Grafana dashboard with multiple types of visualisations](/assets/images/week-03d.png){:.original-size}

### Sections
- [Sprint 3: Connecting the IoT hardware prototype to the cloud infrastructure](#sprint-3-connecting-the-iot-hardware-prototype-to-the-cloud-infrastructure)
- [Artefact #1: Soil moisture sensor data visualisation](#artefact-1-soil-moisture-sensor-data-visualisation) + [Reflection](#reflection-on-artefact-1)
- [Artefact #2: Light sensor data visualisation](#artefact-2-light-sensor-data-visualisation) + [Reflection](#reflection-on-artefact-2)
- [Artefact #3: Temperature sensor data visualisation](#artefact-3-temperature-sensor-data-visualisation) + [Reflection](#reflection-on-artefact-3)

## Sprint 3: Connecting the IoT hardware prototype to the cloud infrastructure
This sprint I spent quite a bit of time trying to help a couple of my team mates get their own cloud infrastructure set up. They wanted to be able to set it up on their own, so they followed the same tutorial I used. For some reason they seemed to run into more issues than I did though; they got error messages that I didn't get, so I found it challenging to try to help them resolve those issues. It didn't help that, like me, they didn't have Linux and Docker experience either. In the end, I was only able to successfully help one of them. It then turned out that we *had* to share one database anyway, so we're using the one I set up as it is already running. I don't think they would have any issues connecting to it, instead the main problems would likely have to do with the format of their MQTT messages as it has to follow the InfluxDB line protocol that seems to be quite pedantic. I hope they've checked out the link I posted in our Teams channel.

Tasks:
- [x] Connect soil moisture sensor and visualise its data
- [x] Connect light sensor and visualise its data
- [x] Connect temperature sensor and visualise its data
- [x] Map soil moisture readings to percentage
- [x] Install and run Chronograf on the server
- [ ] research connecting to and using different databases within our IoT infrastructure
  - possible to route to multiple influxDB output plugins: connecting to the same URL but passing different tables into different databases: https://github.com/influxdata/telegraf/blob/master/docs/CONFIGURATION.md#metric-filtering
- [ ] Phone app IoT On/Off for real-time viz (subscribe directly to MQTT transmission; bypassing db)
{: style='list-style-type: none'}

## Artefact #1: Soil moisture sensor data visualisation
At this stage, carrying over from Sprint 1, my hardware prototype is still bare-bones and consists of only a capacitive soil moisture sensor. For this milestone artefact, the goal is just to connect the sensor to the infrastructure and get the data to be visualised in Grafana; i.e. have the device connect to the MQTT broker (Mosquitto) that I set up on my server and send over the sensor readings.

### Planning
According to [the documentation](https://docs.influxdata.com/influxdb/v1.7/introduction/getting-started/#writing-and-exploring-data), the InfluxDB line protocol (`INSERT` statement format) consists of:
- time(stamp) - optional; automatically added
- measurement - conceptually equivalent to the SQL table
- field e.g. `value=0.62,temperature=27.8` - must have at least one; non-indexed column
- tags - metadata; can have zero to many; indexed column

Considering that the team may have to share the one database, I thought I'd use my name as the measurement (table name). I would have `pot= ` as a tag (metadata) to identify individual pots, and the field would be `soil_moisture= `. The MQTT message to publish, that would effectively be my database `INSERT` statement, would therefore be: `catt,pot=1 soil_moisture= `.

### Execution
I tried combining the relevant bits of my last working code from Sprint 1 with Daniel's code that worked in Sprint 2. There were many lines that I assumed were specific to his sensors so I looked up the [original code template](https://platformio.org/lib/show/5956/EspMQTTClient) that he used, and experimented with it. I tried outputting to the Serial Monitor first in order to check the sensor readings, but it was showing gibberish. Looking it up, this turns out to be an issue with the monitor speed, and it was resolved by adding the line `monitor_speed = 115200` to the project configuration file.  

The device was able to connect to Mosquitto successfully, but the sensor readings seemed to be stuck at the value 4095 whether it was dry or dipped in water... I remembered then that I'd dropped it in class, and wondered whether I'd broken it :( I decided to try running my Sprint 1 code that connected to the Blynk app on my phone and connected the sensor to a graph visualisation so I could see whether it would change over time, but it didn't work either...until I realised that I was using a different pin for that project: analog pin 39, compared to analog pin 2 for the current project, and when I connected the sensor to 39 instead, it started working like it did in Sprint 1 (yay!).  

All that's left for me to do was to change the pin I was reading from in my current project, print the sensor readings to the Serial Monitor to confirm that it's working as intended, then make small changes to the code to publish the data to Mosquitto, finally set up a graph for it in Grafana, and...voila!

![Graph of soil moisture sensor readings in Grafana](/assets/images/week-03a.jpg){:.original-size}

## Reflection on Artefact #1
My code seems to be a lot simpler than Daniel's, which I'm glad about. I wonder if it's got to do with us using different sensors. I guess I'll find out as I add more sensors to the mix. I'd like to add a light sensor next.  

While setting up the graph on Grafana I realised that I might eventually need to tweak my MQTT message (database `INSERT` statement). I don't think I've thought out the data structure well enough just yet. I haven't fully considered what my data needs might be down the track and the different ways I might want to query the data. I can think of a couple of ways to structure it:

1. `catt,pot=1,measure=soil_moisture value= `
2. `catt,pot=1 soil_moisture= `

I've gone with option 2 at the moment, as this way I can add more sensors to the one message.

## Artefact #2: Light sensor data visualisation
After Artefact #1 was successfully set up, my next step was to add a light sensor to the mix, and hopefully be able to pipe the readings through to Grafana like I did with the soil moisture data.

### Initial research
I first looked up the light sensor that I got from Danon, and it appears to be the [Adafruit GA1A12S202 Log-scale Analog Light Sensor](https://www.adafruit.com/product/1384). Conveniently there is also a [getting started guide](https://cdn-learn.adafruit.com/downloads/pdf/adafruit-ga1a12s202-log-scale-analog-light-sensor.pdf) for it that shows how to wire and program it with a sample sketch. This is geared towards Arduino boards but I thought I'd just give it a try on my ESP32 board and see how it goes.

### Planning
For the addition of the light sensor, I wanted to send the data all in one go; each MQTT message published would contain all the data for the pot. To achieve this, I added a `light= ` field to the existing message format: `catt,pot=1 soil_moisture= ,light= `.

### Connecting the light sensor
Following the guide linked above, the wiring seemed quite straightforward. My main hurdle at this point was my lack of understanding of the hardware: I did not really know what difference it would make whether I connect the VCC to the 5v pin or the 3.3v pin. When I read up on it apparently it affects the accuracy of the sensor. The guide instructs to connect the VCC to the 5v pin, then 3.3v to the AREF on the Arduino board, but I am working with an ESP32 which does not have any AREF pin. I ended up trying the VCC on both the 5v and 3.3v pins, and it seemed there was a difference in the readings. Measuring the brightness of my phone flashlight, at 3.3v I was getting readings between 1000 to 2000 lux, while at 5v the readings were mostly between 3000 to 4000 lux.

### Coding for the light sensor
I started a new PlatformIO project in order to have a clean slate from which I can try out the sample sketch in the getting started guide. It was short and straightforward, so once I confirmed the sensor was connected and functional, I added the code to my prototype codebase. It was starting to become unwieldy to be building the MQTT message by appending strings however; I wondered whether there was a string builder function I could use instead. Then, while trying to help Malan with her code I happened to see that she was using `snprintf()`. It turned out to be a buffer which I then used to build my message before publishing it. I think this function makes my code more readable.

Danon also advised me to use the BlynkTimer library instead of the delay() I'd been using. Luckily for me I had used it in the Sprint 1 code that he walked me through, so it was relatively easy to replicate the usage in my current codebase. To be honest I don't think I notice any difference, but **<- research delay()**

Adding the light sensor was a relatively smooth and easy process thanks to the guide. All that was left was to set up a new visualisation panel on the dashboard.

![Dashboard with soil moisture data and light data graph panels.](/assets/images/week-03b.jpg){:.original-size}

## Reflection on Artefact #2
I was a bit torn between feeling like I should slow down and try to have a deeper understanding of the pins, and the impatience of wanting to move on to my next tasks. The former felt like it could potentially be a rabbit hole though; I ended up going with the 3.3v pin which in hindsight seemed to be the one compatible with the next sensor anyway.

## Artefact #3: Temperature sensor data visualisation

### Research and preparation
Initially, I thought that the sensor that I got was a [DS18B20 waterproof temperature sensor probe](https://www.banggood.com/2-Wires-Cables-Digital-Temperature-Sensor-Probe-DS18B20-Thermometer-Thermal-p-986819.html?ID=44707&cur_warehouse=CN), but it turned out to be a [NTC thermistor](https://www.e-tinkers.com/2019/10/using-a-thermistor-with-arduino-and-unexpected-esp32-adc-non-linearity/) instead. I couldn't have figured this out without Danon's help. He helped me connect and test it too. We eventually decided that it wasn't behaving as we expected, however; since I wanted to measure environmental temperature instead of soil temperature, he gave me a different sensor to use: a [BME280](https://learn.adafruit.com/adafruit-bme280-humidity-barometric-pressure-temperature-sensor-breakout/arduino-test). I soldered it myself under Danon's guidance :)

![Me soldering the BME280](/assets/images/week-03c.png)

### Connecting the temperature sensor


## Reflection on Artefact #3

## Artefact #4: Database web UI setup: Chronograf
