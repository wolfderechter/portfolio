---
title: "carbo - co2 meter from scratch"
layout: ../../layouts/WritingLayout.astro
---

## Introduction

As someone who works indoors for long periods of time, the emerging CO2 awareness of these past years has been interesting to follow. I frequently try to air out my workspace, but that is not always possible.

The industry standard of the CO2 meters at the moment is the aranet 4 with a steep price tag of ~200 euro. Which seemed to be a bit too much. After some discussions with a friend we decided to try making our own from scratch.

Which was perfect for me since my exposure to embedded development was limited to making an LED blink with an arduino years ago. So naturally this was the perfect opportunity to see what that area of software development is all about and dive a little deeper. Full code is available on [github](https://github.com/wolfderechter/carbo).

## Hardware

There are a few different CO2 sensors out there, all with their own pros and cons but we ended up choosing the SCD41. This sensor reads CO2, temperature and humidity every 5 seconds which we bumped to 15 seconds to save on resources. Paired with an ESP32 microcontroller board. I chose to use an eink display, while my friend opted for an LCD screen. This was especially interesting so we could really see the differences in code and real life. We chose a modular approach for our shared codebase to easily switch between these two types of displays.

<div style="display: flex; justify-content: space-evenly;">
  <img style="width: 40%" src="/carbo/eink_screen.png" alt="eink screen">
  <img style="width: 40%" src="/carbo/lcd_screen.png" alt="lcd screen">
</div>

## First implementation

We first utilized the ESP32 as the all-in-one solution, having a dashboard with latest values and a 24h chart running on the device itself. Getting the latest values with websockets. This was pretty cool but the drawback was that you'd have to know the device's IP address before being able to visit the dashboard and viewing the values. And because of the limited RAM of the ESP, we'd only be able to store ~24h of CO2 values with an average of 5 minutes.

<img style="margin: auto; display: block; width: 50%" src="/carbo/webpage_v1.png" alt="first implementation of the webpage">

## Going fullstack

So we continued development and chose to implement a universal backend, written in NodeJS paired with an InfluxDB database. We put the setup in a docker compose config and selfhosted it.

```yml
services:
  backend:
    build:
      context: .
    restart: always
    env_file: .env
    environment:
      - NODE_ENV=production
    ports:
      - "3000:3000"
  influxdb:
    image: influxdb:latest
    volumes:
      - ./influxdb/data:/var/lib/influxdb2:rw
    env_file: .env
    ports:
      - 8086:8086
    restart: unless-stopped
```

This was the perfect solution for us since we can now visit the built-in influxdb webpage and query precisely our data

<img style="width: 100%" src="/carbo/influxdb.png" alt="influxdb dashboard example">

## Bill of materials (BOM)

For the eink setup the bill of materials ends up as:

| Name  | Description                 | Price (€) |
| ----- | --------------------------- | --------- |
| SCD41 | CO2 sensor                  | 20        |
| eink  | E-Ink display               | 5         |
| ESP32 | Microcontroller             | 3         |
| Wires | wires to connect everything | 3         |
| Total |                             | 31        |
