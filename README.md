# PUMP for Digitakt

An SP-303 Vinyl Sim-style master compressor for the **Elektron Digitakt mk1**, as an
[elekloader](https://github.com/irpina/elekloader) mod (id `pump-dt`), in place of the Digitakt's stock
compressor. A levelling compressor on everything the Digitakt plays: turn it on and the mix comes up
louder, denser and glued, with a pump that breathes with the beat. It takes the Compressor's page among the
master pages and is on every output: main outs, headphones and USB.

By Robbie (designed and tested on a Digitakt mk1, OS 1.54). GPL-2.0-or-later.

**This is the Digitakt version.** For the **Digitone mk1**, use [PUMP for Digitone](https://github.com/ClaidArremer/digitone-pump)
instead: the same compressor and page, on a master page of its own. The files here do not run on a Digitone,
and the Digitone's do not run on a Digitakt.

![The PUMP page (drawn here on the Digitone; the Digitakt's is the same)](docs/pump-page.gif)

> Unofficial and unsupported. Flashing modified firmware is at your own risk; the stock Digitakt OS always
> recovers the unit (send the stock .syx from the boot menu).

## What you need

| | |
|---|---|
| Unit | **Digitakt mk1** (not the Digitakt II) |
| OS | **1.54** (tested). A 1.53 build is included, untested |
| Mods | elekloader's Digitakt **core 4.1** (its master output and master pages), included here in `elemods/` (the elekloader app does not ship 4.1 yet) |

## Install

In the elekloader app, add both files for your OS from `elemods/`: `core-4.1-os1.54.elemod` and
`pump-dt-1.3-os1.54.elemod` (or the two `-os1.53` files), build, and send the firmware to the Digitakt.
From the command line:

```
python -m elekloader.patch --stock Digitakt_OS1.54.syx \
  --mod elemods/core-4.1-os1.54.elemod --mod elemods/pump-dt-1.3-os1.54.elemod --out Digitakt_pump.syx
```

It combines with digihealth (its FAST AUDIO too), Digi Mono, DH-101 and machine-pages, with the LØST LVLS
boot splash, and with [DIGISNAKE for Digitakt](https://github.com/ClaidArremer/digitakt-digisnake) (tested together on a unit).

## In place of the stock compressor

- **The page**: PUMP takes the Compressor page's place among the master pages (FUNC + LFO): PUMP, Internal
  Mixer, External Mixer.
- **The sound**: the stock compressor and its sidechain no longer run. PUMP works on the master pair before
  it goes to the main outs, the headphones and the bus the USB stream is built from.
- **The settings** live in the compressor's eight parameters, so they are kept with each pattern, saved
  with the project and kept over a power-off, as the stock compressor's were. With SETTINGS > GLOBAL
  FX/MIX making the compressor global, every pattern shares one set, as the stock's did.
- A pattern that never used the compressor starts with PUMP off; switching it on fills in PUMP's
  defaults. A pattern whose compressor had a sidechain source set opens with PUMP on, playing its old
  compressor values as PUMP's: turn it off, or CLEAR the page.

## The page

**FUNC + LFO** steps through the master pages. The knobs sit as on the page: A–D on the top row, E–H below.

| Knob | Name | What it does |
|---|---|---|
| A | PMP | PUMP on or off (a smooth fade; off, it costs nothing) |
| B | ATK | Punch: each hit's first 0–15 ms gets through before the gain comes down |
| C | OUT | Output level before the limiter: −32 to +6 dB, 0 dB in the middle. Turn down for headroom |
| D | RAT | Ratio, in eight steps: AUTO (CMP sets it), 2:1, 3:1, 4:1, 6:1, 10:1, 20:1, LIMIT |
| E | CMP | Compression: threshold and make-up (and the ratio, with RAT on AUTO). More is always louder and denser |
| F | REL | Release: LEVEL (slow, smooth) through GLUE to PUMP (fast, breathing) |
| G | WRM | Warmth: drive into a soft saturator, peaks held. Louder and grittier as it rises |
| H | MIX | Parallel blend of the dry and compressed sound |

- **Turn**: one step a click. **Press a knob and turn**: moves from multiple to multiple of 8 (61, 64, 72 ...).
  **FUNC + turn**: jump to an end.
- **YES**: swap the turning record for a graph of the last 2.8 s of gain reduction, and back.
- The record turns once a bar at the sequencer's tempo and stops with it; the tonearm rides in with the
  gain reduction. The **GR** meter shows it in 2 dB segments with a peak tick. 20 frames a second.
- **CLEAR** (FUNC + held LFO + PLAY): PUMP's defaults, PUMP off.
- **SETTINGS > GLOBAL FX/MIX > PUMP ANIMATION**: untick it for a still page (the controls only, no record,
  graph or meter).
- The Digitakt shows no drop-down messages here; the values are on the page.

Always on while PUMP is: a 30 Hz high-pass in the compressor's detector, so sub-bass you cannot hear does
not drive the pump (the output keeps it), and a soft limiter after OUT that holds peaks under 0 dBFS.

## CPU

Measured on a unit with SYSTEM INFO (irpina's digihealth): all eight tracks playing, the sends and PUMP on,
about 74% CPU and 71–78% DSP. The stock compressor's share goes to PUMP.

## How it works

- **The DSP** (`src/pump.c`, built with `-DPUMP_DT`) is the Digitone's, once a 32-frame block (1500 a
  second) on core 4.1's `ev_render_master`, in place on the master pair. The gain computer works in a log2
  domain from two small tables, with a fast stage (attack 0.5 ms, release 800 ms to 70 ms) and a slow,
  programme-dependent leveller; then WRM, OUT and the ceiling. No libc, no FPU, no 64-bit products.
- **The compressor**: its first instruction (`0x4007425e`, the same in 1.53 and 1.54) jumps to
  `digipump_comp`, which passes the mix through to the compressor's output buffer (the stage after it
  builds the master from that and the reverb's return) and reads the eight parameters the render hands
  the compressor, for PUMP.
- **The page** (`src/pumpdt.c`) is a core 4.1 master page placed after the Compressor's, which it then
  takes out of the list. Its drawing is the Digitone's. Edits go through the compressor page's own edit
  routine (the master view's slot 22), so the firmware clamps, stores and saves them. The record turns,
  and the page redraws, by the audio blocks.
- **The SETTINGS row** is `src/pumpdtset.s`, on core 4.1's `ev_fxmix`.

| | 1.53 | 1.54 |
|---|---|---|
| the compressor | `0x4007425e` | `0x4007425e` |
| its page's edit | `0x400388f6` | `0x400388f6` |
| the transport, the tempo | `0x4199dc2c`, `0x4020db60` | `0x4199ec2c`, `0x4020df10` |

## Building

With elekloader and an m68k cross GCC on the path:

```
python -m elekloader.sdk.build --stock Digitakt_OS1.54.syx .
```

## Credits and third-party code

PUMP is Robbie's own work, written with Claude (Anthropic), except for one part adapted from another
GPL-2.0-or-later project:

| File | Origin | Licence |
|---|---|---|
| `src/pumpdtset.s`: the SETTINGS row's four menu callbacks | Adapted from **digihealth**'s SYSTEM INFO row by irpina ([irpina/digihealth](https://github.com/irpina/digihealth)) | GPL-2.0-or-later |

Everything else is new. The algorithms used are standard textbook ones: a Chamberlin state-variable
filter, Bresenham's line, fixed-point integer maths.

PUMP builds against irpina's [elekloader](https://github.com/irpina/elekloader) SDK headers and runs on its
core (GPL-2.0-or-later). The two `elemods/core-4.1-*.elemod` files are unmodified builds of elekloader's
`mods/core`, made from
[its source at commit be1d095](https://github.com/irpina/elekloader/tree/be1d095cc616e53d14ebf4039c3d297319e1120a/mods/core),
because the elekloader app does not ship core 4.1 yet; they carry only that mod's code and no Elektron code.
The firmware addresses in `mod.json` and the sources were found by studying the Digitakt's own OS so the mod
can work with it; no Elektron code is included. Inspired by the sound of the **Roland SP-303 Vinyl Sim**
compressor and Kwazi's free Ableton rack take on it; no code from either.

## Trademarks

Elektron and Digitakt are trademarks of Elektron Music Machines; Roland and SP-303 of Roland Corporation;
Ableton of Ableton AG. They are named only to say what this works with or was inspired by. This project is
not affiliated with or endorsed by any of them.

## Licence

GPL-2.0-or-later, see [LICENSE](LICENSE).
