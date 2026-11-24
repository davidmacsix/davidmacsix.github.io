---
layout: splash
title: "macsix.io"
permalink: /
header:
  overlay_image: /assets/images/projects/floorplan/header.jpg
  overlay_filter: 0.45
  caption: "My house as a live Home Assistant dashboard"
  actions:
    - label: "See how it was built"
      url: /projects/3d-floorplan-dashboard/
excerpt: >-
  Things I've built for fun: home automation, energy monitoring and a few rabbit holes along the way.
intro:
  - excerpt: >-
      Hi, I'm David. This is where I write up my side projects: what I built, how it works and what tripped me up,
      in enough detail that you could build your own.
feature_row:
  - image_path: /assets/images/projects/floorplan/teaser.jpg
    alt: "Isometric render of the house with live temperatures and lights"
    title: "A 3D floorplan dashboard for Home Assistant"
    excerpt: "A Blender render of the house that lights up, warms up and charges up in step with the real thing."
    url: /projects/3d-floorplan-dashboard/
    btn_label: "Read more"
    btn_class: "btn--primary"
  - image_path: /assets/images/projects/inverter/teaser.jpg
    alt: "Solar, battery and grid power flows on the dashboard"
    title: "Connecting a Duracell Energy inverter to Home Assistant"
    excerpt: "Reading and controlling the inverter locally over Modbus, with no cloud app in the loop."
    url: /projects/duracell-inverter-home-assistant/
    btn_label: "Read more"
    btn_class: "btn--primary"
  - image_path: /assets/images/projects/ev-charger/teaser.jpg
    alt: "The car charging on the floorplan dashboard"
    title: "Freeing a Duracell DuraCharger with OCPP"
    excerpt: "Ditching the vendor cloud and charging the car from spare solar."
    url: /projects/duracharger-ocpp-home-assistant/
    btn_label: "Read more"
    btn_class: "btn--primary"
  - image_path: /assets/images/projects/network/teaser.svg
    alt: "Fibre into an SFP+ module, out to Wi-Fi 7 access points"
    title: "A UniFi network without the Virgin Media Hub 5x"
    excerpt: "An SFP+ ONT stick in place of the hub, Wi-Fi 7 and the full 2 Gbps."
    url: /projects/unifi-network-virgin-media/
    btn_label: "Read more"
    btn_class: "btn--primary"
  - image_path: /assets/images/projects/santa-claude/teaser.svg
    alt: "Four equally sized presents with level budget bars"
    title: "Santa-Claude"
    excerpt: "Using Claude Code to run Christmas for four kids: wishlists, fair budgets and Black Friday price watches."
    url: /projects/santa-claude/
    btn_label: "Read more"
    btn_class: "btn--primary"
---

{% include feature_row id="intro" type="center" %}

{% include feature_row %}
