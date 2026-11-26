---
title: "A 3D floorplan dashboard for Home Assistant"
permalink: /projects/3d-floorplan-dashboard/
excerpt: "A Blender render of the house that lights up, warms up and charges up in step with the real thing."
categories: [projects]
tags: [home-assistant, blender, python, dashboard]
header:
  overlay_image: /assets/images/projects/floorplan/header.jpg
  overlay_filter: 0.5
  teaser: /assets/images/projects/floorplan/teaser.jpg
---

Most Home Assistant dashboards are grids of tiles. They work, but they don't feel like *your house*. I wanted to open
one screen and see the whole place at a glance: which lights are on, which rooms are heating, what the solar panels
and battery are doing, and whether the car is plugged in. So I built a 3D model of the house and made it live.

![The finished dashboard on a winter evening]({{ "/assets/images/projects/floorplan/evening.jpg" | relative_url }})
*An evening view. Lights glow where they're actually on, orange outlines mark the rooms that are heating, and the
temperature, humidity and power for each room float just above its outer wall.*

## What it shows

Everything on the picture is driven by live Home Assistant state:

- **Lights.** Every light in the house glows in its real position: ceiling spots, lamps, a mirror with bulbs round it,
  even the light inside a cupboard. Turn two on in the same room and their glows add up, just like real light.
- **Heating.** Each room's temperature floats just above its outer wall. The colour runs from blue (cold) through green to
  orange-red (warm), and a glowing orange line traces round the top of the walls while that room's radiator valve is open.
- **Presence.** A thinner white line runs round the en-suite while its mmWave presence sensor sees someone.
- **Energy.** The solar panels glow brighter as output rises, the battery cabinet shows its charge level, and animated
  dashes flow between the panels, battery, house, grid and EV charger.
- **The car.** When the car is home it appears on the drive, with its charge level beside it, and a cyan line runs to the
  charger while it's charging.
- **Blinds.** A roller blind is drawn as a live SVG and drops smoothly as the real one closes.

Tap any room, including its walls, and a pop-up opens with that room's lights, thermostat, blind and power use.

![Solar, battery and EV charging on a sunny afternoon]({{ "/assets/images/projects/floorplan/sunny-charging.jpg" | relative_url }})
*A sunny afternoon: 4.2 kW of solar charging the battery and the car, with a little going out to the grid.*

## How it works

The trick is that **nothing is rendered live**. Blender renders everything once, ahead of time, as a stack of
transparent images. Home Assistant's built-in
[picture-elements card](https://www.home-assistant.io/dashboards/picture-elements/) then stacks those images on top of each
other and shows or hides each one depending on an entity's state.

### 1. The model

The house started life on paper. When we bought it, the deeds came with a floorplan, so I scanned that and imported it
into [Sweet Home 3D](https://www.sweethome3d.com/) as a background image. Tracing over the scan gave me the walls,
doors and windows of the initial model.

The deeds only showed the house as it was when we bought it, though. Since then we've moved walls, taken out hallways
and created new rooms, so I edited the traced model to match the house as it is today.

Sweet Home 3D is great for laying out a house but not for rendering it the way I wanted, so I exported the model and
imported it into [Blender](https://www.blender.org/). There I cut the walls down to just above waist height, so you can
see into every room from an isometric camera, and started furnishing it.

Later rooms were furnished one at a time, working from photos of each wall: desks, monitors on arms, beds, a swivel
chair, wall colours. Where I could, furniture is modelled to the manufacturer's published dimensions so it sits right
in the room.

![A bedroom modelled from photos]({{ "/assets/images/projects/floorplan/room-from-photos.jpg" | relative_url }})
*One bedroom, modelled from four photos. The walls are painted to match and the furniture is built to IKEA's published sizes.*

Once the dashboard was up and running, one rule kept it manageable: **the .blend file is no longer edited by hand.**
Every change (opening up a cupboard, cutting a wall down, adding furniture, repainting a room) is a Python script that
runs against the model in memory at render time. That means every tweak is repeatable, reviewable and easy to undo.

### 2. Rendering the layers

A headless Blender run (`blender -b house.blend --python render.py`) produces:

- **A base image** with every light off, lit only by a soft overhead area light.

  ![The base render]({{ "/assets/images/projects/floorplan/base-render.jpg" | relative_url }})

- **One image per light**, containing *only* that light's contribution. Each light is rendered on its own and the base
  is subtracted, leaving just the glow. In the browser these are blended with CSS `mix-blend-mode: screen`, so any
  combination of lights adds up correctly without rendering every combination.
- **Energy states**: solar panels at three brightness levels, the battery at five charge levels, charger LEDs in three
  colours, and the car (with and without the balcony lights on it).
- **Room outlines**: each room's outline rendered as a thin emissive tube while everything else in the scene is a
  holdout. The walls still hide the parts of the line they should, but the image contains nothing except the line.

Renders are done at 3840×2160 with Cycles and take around 20 minutes for the full set on a MacBook Pro. Lights that
should stay in one room need help: Cycles light happily leaks through the cut-off walls, so the scene has invisible
"blocker" boxes that extend each wall to full height for lighting but don't appear in the camera.

### 3. Tracing the rooms

The outlines (and the tap areas, which use the same shapes) are generated, not drawn. A script fires rays straight down
through the model to build a height map of the wall tops. Another closes each doorway by pairing up the door posts on
either side, labels the enclosed floor areas as rooms, and traces the contour of each one. Every point is then lifted to
the height of the nearest wall top, so the line runs along the top of full walls, steps down over half walls and goes
round door frames.

### 4. Putting it on the dashboard

The camera is an orthographic isometric view, which makes the maths pleasant: any point in the model maps to the
screen with a simple affine transform:

```
screen = P0 + x·ux + y·uy + z·uz
```

That one formula positions everything that isn't a pre-rendered image:

- **Labels** are `state-label` elements with a CSS `matrix()` transform that lays the text in a wall's plane. Each room's
  temperature, humidity and power float just above its outer wall, so they line up with the house without covering it.
- **Tap areas** are invisible images clipped to each room's outline with CSS `clip-path`, stacked back to front so the
  nearer room wins where two rooms share a wall.
- **The blind** is a small custom element that draws an SVG in the window's wall plane, using the same projection as an
  SVG `matrix`, and animates the fabric with a CSS transition as the cover's `current_position` changes.

![The live blind at 100%, 45% and 0% open]({{ "/assets/images/projects/floorplan/blind-states.jpg" | relative_url }})
*The blind is an SVG, not a render, so it can sit at any position and animate between them.*

There are two small custom elements, one for the blind and one for the temperature labels (which change colour and pulse
while heating). Both are registered as dashboard resources through `data:` URLs, so there's nothing to host. The room
pop-ups use [Bubble Card](https://github.com/Clooos/Bubble-Card) from HACS.

A Python script assembles the whole card: it uploads the images through Home Assistant's image upload API, builds over
two hundred elements and conditions, and saves the dashboard over the WebSocket API. A second script renders a preview of
the card from live (or made-up) states, so I can check a change before it goes anywhere near the real dashboard. All the
screenshots on this page come from that preview.

## Things that tripped me up

- **Light bleeding between rooms.** With the walls cut low, a lamp in one bedroom lit the ceiling-less room next door.
  The invisible full-height blockers fixed it, but they have to be updated every time a wall changes.
- **Rotated objects in background mode.** Some of the blockers are rotated, and in headless Blender their
  `matrix_world` came back as identity. Reading the location and rotation directly was the reliable route.
- **A stale background.** After opening up a cupboard I couldn't see the change on the dashboard. The build had been
  rebuilding every layer except the background JPEG, which hadn't changed for days.
- **Things hung on walls.** Pictures, mirrors and shelves that hang above the wall cut would float in mid-air, so they're
  left out.
- **iCloud.** The project lives in an iCloud-synced folder, and macOS sometimes evicts the renders to the cloud.
  `brctl download` before reading them saves some confusing errors.

## How it evolved

The first version had coloured boxes for tap areas and dots for lights. It worked, but it looked like a diagram.

![An early version with coloured tap areas]({{ "/assets/images/projects/floorplan/early-tap-areas.jpg" | relative_url }})

Over a couple of weeks of evenings it gained per-light glows, HD renders with warmer lighting, energy flows, the car,
room outlines, furniture and live blinds. I built it with [Claude Code](https://claude.com/claude-code) as a pair
programmer. I described what I wanted, often with photos of a room, and we iterated on previews until it looked right.

## Want to build one?

You'll need:

- A 3D model of your home in [Blender](https://www.blender.org/). A scanned floorplan traced in
  [Sweet Home 3D](https://www.sweethome3d.com/) and exported to Blender is a quick way to get one.
- An orthographic camera, and the discipline to keep all your changes in scripts.
- Home Assistant's picture-elements card, plus [Bubble Card](https://github.com/Clooos/Bubble-Card) if you want pop-ups.
- Python with Pillow and NumPy to post-process the layers, and the Home Assistant WebSocket API to publish the card.

Start small: a base render, one light layer with `mix-blend-mode: screen`, and one state label. Once those line up,
everything else is more of the same.
