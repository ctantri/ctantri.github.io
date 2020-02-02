---
layout: post
title: "Submission 2"
date: 2020-02-03
---
![Screenshot of Grafana dashboard after successful infrastructure setup](/assets/images/week02c.jpg){:.original-size}

### Sections:
- [Sprint 2: IoT cloud infrastructure](#sprint-2-iot-cloud-infrastructure)
- [Artefact #1: Database setup: InfluxDB](#artefact-1-database-setup-influxdb) + [Reflection](#reflection-on-artefact-1)
- [Artefact #2: MQTT broker setup: Mosquitto](#artefact-2-mqtt-broker-setup-mosquitto) + [Reflection](#reflection-on-artefact-2)

## Sprint 2: IoT cloud infrastructure
For this sprint our task is to set up an infrastructure in the cloud to receive sensor data, store it in a database, then retrieve and visualise that data. I was assigned the task of installing the database for this infrastructure, but when I finished it I realised there wasn't much action to be had with a database on its own, so with some time left over I went on to experiment with building the rest of the infrastructure. The tutorial I followed can be found [here](https://dzone.com/articles/raspberry-pi-iot-sensors-influxdb-mqtt-and-grafana).  

I would like to note that as someone who is not familiar enough with Linux CLI (or just CLI in general), I experienced hiccups along the way, the first one being that I didn't know how to move on and install the next infrastructure component while the previous one was running (I thought I had to stop the process, in which case how do I get them all to run at the same time?). Each one is in a separate Docker container anyway, so my problem had to do with working within the one terminal window I'm remotely connected to.  

This is where I remembered something that I learnt from a data scientist I had worked with in the past: [Linux Screen](https://linuxize.com/post/how-to-use-linux-screen/). It lets me run each Docker container inside its own screen that I can attach to and detach from as needed, and leave them running even when I'm not connected. I like this setup as it keeps things organised (tip: make sure to name each screen).

Another CLI-related hiccup had to do with my limited experience with text-editing within the terminal window. I only know of vi/vim, so that was what I used, though I had to look up a [cheat sheet](https://www.linuxtrainingacademy.com/vim-cheat-sheet/) for it because I last used it in 2017 in *31268 Web Systems* :)

I also learnt about configuration files - that settings that have defaults tend to be commented out in the files, so if we are changing them we need to also uncomment those lines. This seems obvious but for some reason it did not automatically click for me, perhaps because the first configuration file I encountered was mostly commented out, so it seemed to me like that was how it was supposed to be.

## Artefact #1: Database setup: InfluxDB
A popular time-series database that kept coming up in my research was InfluxDB. There are three versions of this database, namely InfluxDB v1.x, InfluxDB v2 and InfluxDB Cloud. The Cloud version provides database as a service at a fee, and while InfluxDB v2 provides more features, such as a built-in visualisation engine, it is currently in beta. For this learning project I decided to use InfluxDB v1.x as it is a stable product that is also open source and free. I ended up installing it twice: once directly on the server, then a second time in a Docker container following the tutorial linked above.

### First installation
I spun up an Ubuntu server to try things out. I started off with the [installation instructions](https://docs.influxdata.com/influxdb/v1.7/introduction/installation/), followed by the [getting started guide](https://docs.influxdata.com/influxdb/v1.7/introduction/getting-started/). The process was rather straightforward, culminating in `CREATE DATABASE mydb` then  a couple of `INSERT` and `SELECT` statements. One thing I noticed was that InfluxDB had its own syntax for `INSERT` statements that needs to be followed strictly for a successful operation - refer to [this explanation](https://docs.influxdata.com/influxdb/v1.7/introduction/getting-started/#writing-and-exploring-data) in their documentation.  

![Screenshot of terminal window after database setup](/assets/images/week02a.jpg){:.original-size}

### Second installation
I installed InfluxDB in a Docker container. This is the second step in the tutorial, after installing and running Mosquitto. This time the installation itself is a breeze - literally just one line. From there it is a matter of a few more lines to create a new database and set up a user for Telegraf to connect with. This was also really straightforward, except that the statement to set up the user should really be `CREATE USER telegraf WITH PASSWORD 'telegraf'`. Double quotes on the password, as given in the tutorial, did not seem to work.

## Reflection on Artefact #1
Given that sensor data tends to be time-series data, I am inclined to try out a time-series database for this prototype. While we can store time-series data in a relational database, my understanding is that time-series databases are specifically geared towards time-series data with automatic timestamping (as an automatic insert into a timestamp data field), automatic timestamp-indexing and presumably more powerful time-based query options.  

A hiccup I had during the set up process had to do with my lack of experience with the command line in the first place: I could not tell whether the database was already running or not, so I ended up quite confused until Danon pointed out that it was a daemon process already running in the background. Now I know how to check its status :)

Also, I must admit that I have yet to fully understand the InfluxDB `INSERT` statement syntax; I've only mimicked and tweaked the examples given. I think that I will need to dive into it going forward: it seems that the team will be sharing the one database, so we'll need to decide on some sort of structure to keep our data separate in the NoSQL database.

## Artefact #2: MQTT broker setup: Mosquitto
Mosquitto is the MQTT broker used in the tutorial. It is also being installed in a Docker container, and the installation and setup instructions provided are very simple and straighforward (one line each!). Unfortunately after testing it turned out that this did not work. The tutorial relied on default configuration values for the ports that Mosquitto would listen on, specifically port 1883. It appeared, however, that port 1883 was blocked by the university's firewall. Danon advised us to use port 5269 instead.

In order to change the configuration, I ran bash in the Docker container where Mosquitto was running, and used vi/vim to edit the mosquitto.conf file while referring to the examples in [this guide](http://www.steves-internet-guide.com/mossquitto-conf-file/). Once the changes were saved, I restarted Mosquitto so that they could take effect.

## Reflection on Artefact #2
The main issue I had with this artefact was not to do with Mosquitto itself, but rather with Docker. I'd been curious about Docker so I'm happy that I got to dive into using it, but I had next to zero previous experience with it. There were several things I learnt while trying to make changes to the Mosquitto configuration file:
- Use `docker ps` to list all containers that are currently running, but it does not include containers that have been stopped. For that, add a flag: `docker ps -a`.
- The `docker run` command creates a new container every time. To reuse an existing stopped container, use `docker start <container name>`.
- Best to make sure to name containers, otherwise they get assigned random names automatically.

If you couldn't tell what issue I had from looking at the above list: I had edited the configuration file in one container, then proceeded to run Mosquitto in a new container without realising it, and got stuck wondering why my changes would not stick :) My breakthrough point came only after I started naming the containers. I didn't realise that stopped containers were not automatically deleted (wasn't aware of the `-a` flag for `docker ps`), so when I tried to re-use a container name Docker told me that it already existed and that was my lightbulb moment. I have since deleted the extra 20+ containers that were created as a byproduct of my troubleshooting efforts. Oops.