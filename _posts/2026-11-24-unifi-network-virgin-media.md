---
title: "A UniFi network on Virgin Media full fibre, without the Hub 5x"
permalink: /projects/unifi-network-virgin-media/
excerpt: "Replacing Virgin Media's Hub 5x with an SFP+ ONT stick in a UniFi Dream Machine Pro, then building out Wi-Fi 7 and multi-gig around it."
categories: [projects]
tags: [networking, unifi, virgin-media, xgs-pon, wifi]
header:
  teaser: /assets/images/projects/network/teaser.svg
---

We're on Virgin Media's 2 Gbps full-fibre service. It's a great line, but it comes with the **Hub 5x**, an all-in-one
fibre modem, router and Wi-Fi box that does none of those jobs especially well, and, unlike Virgin's older hubs, has
**no modem mode**. You can't simply put your own router behind it without ending up with double NAT and two lots of
Wi-Fi fighting each other.

I wanted a proper UniFi network: one place to manage the router, the access points and every device in the house, and
something that Home Assistant could watch. This is how it came together, starting with getting rid of the hub.

![Fibre into the UDM Pro's SFP+ port, then out to two access points, Home Assistant and a wired desk]({{ "/assets/images/projects/network/topology.svg" | relative_url }})

## The kit

| Part | What it does |
|---|---|
| **UniFi Dream Machine Pro** | Router, firewall and UniFi controller. Two 10G SFP+ ports, one for the internet and one for the LAN. |
| **WAS-110 XGS-PON SFP+ stick** with the 8311 community firmware | The fibre "modem", shrunk into a module that plugs straight into the UDM Pro. Replaces the Hub 5x. |
| **2 × UniFi U7 Pro XG** | Wi-Fi 7 access points (2.4, 5 and 6 GHz) with 10G uplinks. One in the middle of the house, one in the office. |
| **10GBASE-T SFP+ module** | Gives the UDM Pro's LAN SFP+ port a copper RJ45 socket for multi-gig. |
| **UniFi PoE+ injector** and a **TP-Link LS105GP** PoE+ switch | Power for the access points, until a proper UniFi PoE switch arrives. |
| **Cat6a S/FTP**, 23 AWG solid copper | The long runs from the plant room to the access points. |

## Bypassing the Hub 5x

Virgin's full-fibre service runs on **XGS-PON**. The hub's fibre side is an ONT (optical network terminal), and what it
really does is prove to Virgin's network that it's an authorised Virgin hub. Replace the ONT with one that presents the
same identity, and Virgin's network can't tell the difference. Everything after that is your own router.

### The stick

The **WAS-110** (sold under a few names, including X-ONU-SFPP) is an XGS-PON ONT in an SFP+ module. The
[8311 community firmware](https://pon.wiki/) gives it a web interface and the settings needed to *masquerade* as another
ISP's ONT. Mine plugs straight into the UDM Pro's SFP+ WAN port.

### What it needs from the hub

The [pon.wiki guide for the Hub 5x](https://pon.wiki/guides/masquerade-as-the-virgin-media-o2-hub-5x-with-the-was-110/)
is the reference here and worth reading in full. In outline:

- **The hub's PON serial number**, printed on the label on the bottom of the Hub 5x. This is the one value that's unique
  to your line, and it's what Virgin's network checks.
- **A handful of identity values** that are the same for every Hub 5x: equipment ID, hardware and software versions, a
  shared "IP host" MAC address and the right MIB file. The guide lists them.
- **Fix VLANs turned off** in the 8311 settings, so the stick passes Virgin's traffic through as Virgin expects.

These go into the stick's 8311 web page (or over SSH), then a reboot. Once the fibre is plugged in, the status page shows
the optical levels and the connection state. You're looking for **"O5.1, Associated"**, which means Virgin's network has
accepted it.

### Fibre connectors

Virgin's fibre ends in an **LC/APC** connector. Check what your stick needs and use the right adapter or patch lead.
And never mix **APC** (green) and **UPC** (blue) connectors: they're cut at different angles and won't mate properly.

### The router side

The stick is a bridge, not a router, so the UDM Pro does the routing. On my setup the UDM Pro's WAN is plain **DHCP**,
with the line's speeds entered as 2 Gbps so UniFi's graphs make sense. The guide also covers cloning the hub's MAC
address onto the router's WAN, which helps if Virgin's DHCP server is still holding a lease for the hub.

### Things to know

- **Keep the hub connected for a while first.** The guide recommends leaving the Hub 5x in service for about a week
  after installation, until Virgin has fully provisioned the line and closed the install.
- **Keep the hub afterwards.** If anything changes on Virgin's side, it's your fallback, and you'll want it if you ever
  need Virgin support.
- **Keep an eye on the stick.** SFP+ ONT sticks run warm. Mine sits around 53 °C, with receive power around −17 dBm and
  zero errors, all comfortably healthy. The 8311 status page shows all of this.
- **It's unsupported.** Virgin won't help you with it. The [8311 community](https://pon.wiki/) is where the help is.

## Getting the full 2 Gbps

With the hub gone, a speed test on a 1G port topped out at about **870 Mbps**, which is about all 1G Ethernet can carry.
To see the whole line I needed multi-gig all the way to the computer.

The UDM Pro's RJ45 LAN ports are all 1G, so the only multi-gig LAN port is the second SFP+ slot. The 10GBASE-T module in
it happily negotiates 2.5G, so with a cheap **2.5G USB Ethernet adapter** on the Mac plugged into that module:

| Test | Download | Upload |
|---|---|---|
| Mac on a 1G port | ~870 Mbps | ~208 Mbps |
| Mac on 2.5G (via the SFP+ module) | **2,089 Mbps** | ~208 Mbps |

That's the full line rate. (Upload is capped at around 200 Mbps on this package.)

Two things that tripped up my testing:

- **Pick the speed test server.** Ookla's auto-selected server under-reported by a few hundred Mbps. Testing against a
  specific, well-connected server with the Ookla CLI (`speedtest -s <server id>`) gave consistent results.
- **Use Ookla's own CLI.** The Python `speedtest-cli` can't keep up at these speeds.

I also have IDS/IPS and Smart Queues turned off on the UDM Pro.

## Wi-Fi 7 with two U7 Pro XGs

Two access points cover the house: one central, one in the office at the far end. Both are Wi-Fi 7 with 10G uplinks, so
the access point's own link to the network isn't the bottleneck.

Leaving everything on auto worked, but a little tuning made a real difference:

| Band | Central AP | Office AP | Width |
|---|---|---|---|
| 2.4 GHz | Channel 1 | Channel 6 | 20 MHz |
| 5 GHz | Channel 149 | Channel 36 | 80 MHz |
| 6 GHz | Auto | Auto | 160 MHz |

- **Keep the two APs apart** on 5 GHz and 2.4 GHz so they don't share airtime.
- **Look at what the neighbours are doing.** The office AP's 2.4 GHz started on channel 11, which UniFi showed was about
  40% busy with neighbouring networks. Moving to channel 6 cleared it.
- **80 MHz on 5 GHz** rather than the 40 MHz that auto had chosen gave the office AP a big jump in throughput.

Standing near an access point, a MacBook gets about **1,300 Mbps** and an iPhone 16 Pro about **875–1,200 Mbps** over
Wi-Fi. About 50 devices are connected on a typical day.

## The cabling lesson: PoE over long shielded runs

The office AP is about 20 m from the plant room over **Cat6a S/FTP** (shielded, 23 AWG solid copper). Before trusting it
I plugged the Mac into the far end: a clean 2.5G link and the full 2 Gbps, so the cable and terminations were good.

Power was another story. The UniFi PoE+ injector powered the AP over that run every time. The TP-Link PoE+ switch
wouldn't, though it powered the same AP over a short patch lead. Later it did start working over the long run, which
points at PoE detection being marginal on a long shielded cable rather than a fault in the cable itself.

If you're running shielded cable: use keystones or plugs rated for its thickness, keep the foil and braid away from the
conductors, and earth the shield at one end only.

## Home Assistant on the network

The UniFi integration in [Home Assistant](https://www.home-assistant.io/) turns the network into something the house can
react to:

- **Presence.** Device trackers for every client mean Home Assistant knows when phones arrive and leave.
- **Alerts.** An automation notifies my phone if an important device has been off the network for three minutes, and
  again when it's back: the EV charger, the heating bridge, the lighting hub, the doorbell and a couple of gateways.
- **Firmware and health.** The UDM Pro and access points show up as devices, with firmware versions and update entities.

## What's next

- **A UniFi Pro XG 10 PoE switch** on the UDM Pro's spare SFP+ port with a 10G DAC cable. Both APs get 10G and PoE
  from one box, the injector and the TP-Link switch go, and the office desk gets a proper 2.5G port.
- **Re-running the fibre** from where Virgin installed it to the plant room, next to everything else.
- **Symmetrical upload.** Virgin offers a symmetrical upload add-on on XGS-PON, which would take upload from about
  200 Mbps to 2 Gbps. I need to check the stick still authenticates afterwards, and keep the Hub 5x handy just in case.
