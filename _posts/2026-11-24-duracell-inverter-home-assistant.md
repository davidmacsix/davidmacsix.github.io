---
title: "Connecting a Duracell Energy inverter to Home Assistant"
permalink: /projects/duracell-inverter-home-assistant/
excerpt: "Reading and controlling the inverter locally over Modbus, with no cloud app in the loop."
categories: [projects]
tags: [home-assistant, energy, modbus, solar, inverter]
header:
  teaser: /assets/images/projects/inverter/teaser.jpg
---

We have solar panels on the roof and a Duracell Energy system in the plant room. At its heart is a hybrid inverter:
it takes the solar panels' output, charges and discharges two 5.12 kWh batteries (10.24 kWh in total), and decides
how much power to draw from or send to the grid. Out of the box the only way to see or control the inverter is the
Duracell Energy app, which talks to it through the manufacturer's cloud. That's fine for a quick look, but it can't be
automated, and I wanted Home Assistant to decide when the battery charges.

This is how I got the inverter talking to Home Assistant directly over the local network, and what I do with it now.

## The key discovery: it's a Skyline inverter

The inverter in a Duracell Energy system is made by **Skyline**. That matters because the Home Assistant community has already done the hard work for Skyline inverters: there's
a custom integration, [HA-Skyline by iPeel](https://github.com/iPeel/HA-Skyline), that reads and writes the inverter's
Modbus registers.

The catch is that the inverter's Wi-Fi and Ethernet only talk to the cloud. The local route is the **RS485 port**, which
speaks Modbus RTU. So the job is to get RS485 onto the network.

## The hardware

![Inverter RS485 port to Waveshare adapter to home network to Home Assistant]({{ "/assets/images/projects/inverter/wiring.svg" | relative_url }})

- **A Waveshare RS485-to-Ethernet adapter.** It's a small box that turns Modbus TCP requests from the network
  into Modbus RTU on the RS485 wire.
- **A short run of Ethernet cable** used as plain wire. One twisted pair goes to RS485 A and B, and another core goes to
  ground (PE). Using a twisted pair for A/B matters: that's what keeps RS485 reliable.
- **A plug for the inverter's RS485 terminal.** The port sits under a small cover on the inverter. If yours didn't come
  with the mating connector, it's a standard 3.5 mm pitch 4-way pluggable terminal block.

Wire A to A and B to B. If you get no response later, swapping A and B is the first thing to try.

## Setting up the adapter

The Waveshare adapter has a small web page for its settings. Out of the box it sits on a fixed `192.168.1.200` address,
so you may need to give your computer an address on that subnet to reach it the first time. Then:

1. Give it a permanent address on your network: either a static IP on the adapter, or a DHCP reservation on your
   router (I use a reservation and a hostname, so Home Assistant connects to the name rather than an IP).
2. Set the **protocol to Modbus TCP to RTU**. This is the important one: in this mode the adapter translates properly
   between the two, rather than just tunnelling raw serial bytes.
3. Set the **TCP port to 502** (the standard Modbus port).
4. Set the serial side to **9600 baud** (I left the other serial settings at their defaults).

## Setting up Home Assistant

1. Install [HACS](https://hacs.xyz/) if you don't already have it.
2. In HACS, add `https://github.com/iPeel/HA-Skyline` as a custom repository of type *Integration*, then download
   **Skyline Inverter** and restart Home Assistant.
3. Go to **Settings → Devices & services → Add integration → Skyline**, and enter the adapter's address. The integration
   scans the Modbus bus and adds the inverter it finds.

That's it. Within a few seconds I had 46 new entities, polled every 10 seconds:

| Type | What you get |
|---|---|
| **Sensors** (34) | Solar power per string (MPPT1/MPPT2), battery state of charge, battery power and voltage, grid power and voltage, house load, daily and lifetime energy for solar, battery and grid, inverter temperature, firmware versions |
| **Numbers** (7) | Grid charge power and target state of charge, battery charge and discharge limits, maximum feed-in power |
| **Select** (1) | The inverter's work mode: Self Consumption Priority, Feed In Priority, Time based Control, Backup Supply, Battery Discharge |
| **Switches** (2) | EPS (backup power) output, and the integration's "match feed-in to excess solar" mode |

The lifetime energy sensors plug straight into Home Assistant's **Energy dashboard** for solar production, battery
charge and discharge, and grid import and export.

![The energy corner of my floorplan dashboard]({{ "/assets/images/projects/inverter/energy-view.jpg" | relative_url }})
*The same sensors drive the energy side of my [3D floorplan dashboard](/projects/3d-floorplan-dashboard/): solar output
per roof, battery charge level and power flowing between the panels, battery, house and grid.*

## Charging on the cheap overnight rate

We're on Octopus Intelligent Go, where electricity is a fraction of the price between 23:30 and 05:30. The aim is
simple: fill the battery overnight at the cheap rate, then run the house from the battery and the panels during the day.

Two small template switches make the work mode easy to use in automations:

- **Charge Battery Mode** sets the work mode to *Backup Supply*. In that mode the inverter charges from the grid up to
  the grid-charge target state of charge.
- **Self Consumption Mode** sets it back to *Self Consumption Priority*, where the battery covers the house's demand
  and solar tops it up.

Then two automations do the work:

- **At 23:30** it sets the grid-charge target from tomorrow's solar forecast (from
  [Forecast.Solar](https://forecast.solar/)), then turns Charge Battery Mode on. On a bright day it doesn't fill the
  battery to the top, leaving room for free solar; in winter it charges to 100%.
- **At 05:30** it turns Self Consumption Mode back on.

```yaml
alias: Stop Charging
triggers:
  - trigger: time
    at: "05:30:00"
actions:
  - action: switch.turn_on
    target:
      entity_id: switch.self_consumption_mode
```

The forecast-based target is calculated by a template sensor (forecast kWh × a confidence factor, never below a minimum),
with an on/off helper to switch the forecast logic off and keep a fixed target.

## A gotcha: the app can still override you

For a few months the battery sometimes charged from the grid at peak rate from 22:00, even though Home Assistant had it
in self-consumption mode. Digging through Home Assistant's long-term statistics showed it happened on 17 evenings,
mostly Mondays.

The cause was a **Scheduled Charge** still enabled in the Duracell Energy app (22:00–05:00 at 4 kW), left over from
before Home Assistant took over. The inverter's own schedule wins over the work mode. It cost about £21 in peak-rate
electricity before I spotted it.

Two lessons:

- **Turn off every schedule in the vendor app** (or set them to 0 W) so that Home Assistant is the only thing in charge.
- **Alert on the unexpected.** I now have an automation that sends a notification to my phone if the battery charges
  from the grid outside 23:30–05:30 for more than three minutes. It ignores charging that solar covers.

## Tips

- **Restarting the inverter.** After a restart, the inverter can briefly report zero for all its lifetime totals,
  which makes a mess of the Energy dashboard. The integration tries to filter that out, but its author recommends
  switching the Modbus adapter off while the inverter restarts.
- **Occasional zero readings.** The inverter sometimes reports a zero for a single poll (you'll see it in the vendor app
  too). The integration averages over 30 seconds to smooth this out.
- **Parallel inverters.** If you have more than one inverter in parallel, read the integration's README: you need one
  Modbus adapter per inverter.

## Was it worth it?

Absolutely. Everything runs locally, so it keeps working when the internet or the vendor's cloud doesn't. The battery
now charges at the cheap rate and only as much as it needs, and the data feeds the Energy dashboard, my floorplan
dashboard and the analysis I used to choose a tariff. Lifetime so far: about 3,860 kWh of solar and over 3,000 kWh
through the battery.
