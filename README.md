# LightSABRE DAC

**An audiophile Raspberry Pi HAT, given away complete.**

Designed by [Dario “Darmur” Murgia](https://github.com/Darmur). Released to the public under [CC0 1.0](LICENSE).

[![License: CC0-1.0](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](https://creativecommons.org/publicdomain/zero/1.0/)
[![KiCad 9](https://img.shields.io/badge/KiCad-9.0-314cb0.svg)](https://www.kicad.org/)
[![ESS Sabre](https://img.shields.io/badge/DAC-ES9018K2M-111111.svg)](https://www.esstech.com/)
[![Raspberry Pi HAT](https://img.shields.io/badge/Form_factor-65%20%C3%97%2056%20mm%20HAT-c51a4a.svg)](#repository-contents)

<p align="center">
  <img src="LightSABRE_DAC.3D-render-03.png" alt="LightSABRE DAC, top view — Rev 1.0 by D. Murgia" width="820">
</p>

Most open DAC projects stop at a schematic sketch and a wish. Darmur shipped the other kind of gift: a finished, manufacturable, dual-edition board with Gerbers, pick-and-place, 3D, and two BOMs that already name LCSC, Mouser, and Digi-Key parts. The work is done. What remains is for someone to make it real, listen, and tell the rest of us what they heard.

This repository is a community fork of [Darmur/lightsabre-dac](https://github.com/Darmur/lightsabre-dac). Nothing here claims his credit. The point of the fork is to keep that gift readable, findable, and easy to build.

---

## Why this board exists

Dario Murgia is an Italian electronics engineer whose public work has always pointed in the same direction: audio hardware that people can actually hold. His GitHub is a trail of Raspberry Pi HATs, Class-D amplifiers, Bluetooth speakers, and a Class-A tube hybrid — boards released so others can learn from them, copy them, and improve them.

LightSABRE is that instinct applied to a proper Sabre DAC.

The name is not decoration. At the centre sits the ESS ES9018K2M, the 32-bit SABRE32 converter that made “Sabre sound” a living-room phrase: HyperStream architecture, on-chip ASRC, and a Time Domain Jitter Eliminator that forgives a Raspberry Pi’s imperfect I²S in a way cheap DAC HATs never do. Around that chip Darmur built the analogue, the clocks, and the power that the datasheet assumes and most boards omit.

He then put his name on the silk — `D. Murgia  Rev1.0` — and waived the copyright.

> Creative Commons CC0 is not a licence with strings. It is a dedication to the commons. You may manufacture LightSABRE, sell LightSABRE, modify LightSABRE, and say so. The only honest thing to keep is the attribution that the designer asked for with a silkscreen line.

---

## What you are looking at

<p align="center">
  <img src="LightSABRE_DAC.3D-render-01.png" alt="LightSABRE DAC, three-quarter view with barrel jack and three 3.5 mm outputs" width="820">
</p>

LightSABRE is a 65 × 56 mm Raspberry Pi HAT. It sits on the 40-pin GPIO, takes I²S from the Pi, and puts analogue music out of three 3.5 mm jacks:

| Jack (silkscreen) | Part | Wiring on the silk |
| --- | --- | --- |
| **Balanced L** | 3.5 mm PJ-320A | Tip = hot (+), ring = cold (−), sleeve = shield |
| **Balanced R** | 3.5 mm PJ-320A | Tip = hot (+), ring = cold (−), sleeve = shield |
| **Unbalanced** | 3.5 mm PJ-320A | Tip = left, ring = right, sleeve = shield |

The same wiring is engraved on the underside, so a late-night cable session does not require the PDF.

<p align="center">
  <img src="LightSABRE_DAC.3D-render-04.png" alt="LightSABRE DAC underside — output pinout and 5.2 V input polarity" width="720">
</p>

A 5.5 × 2.0 mm barrel jack feeds the board. The silk says what the analogue section wants to see: **Vin = 5.2 V**, centre positive. A 5.6 V / 5 W SMB zener sits on that input as a clamp, not as a design suggestion to over-volt it.

A two-pin jumper (`J2` / `JP1`) ties the HAT 5 V rail to the Raspberry Pi. Fit it and the HAT can back-power the Pi, or the Pi can feed the HAT. Leave it off if you want the two 5 V domains independent.

---

## The engineering, in the order it matters

Audiophile boards fail in the power supply more often than they fail in the DAC. LightSABRE is laid out as if Darmur knew that.

### Isolated analogue rails

The 5 V incoming rail does not become the op-amp supply. An isolated 2 W DC/DC module (`A0512S-2WR2`, PS1) creates a galvanically split ±12 V island. Linear 78L09 / 79L09 regulators then drop that island to quiet ±9 V for the analogue stages. The Raspberry Pi’s digital noise stays on its own side of the isolation barrier.

### Split 3.3 V for the Sabre chip

The ES9018K2M is not given a single 3.3 V and a hope.

| Rail | Regulator | Role |
| --- | --- | --- |
| DAC digital 3.3 V | LR6206B-U33 | Digital core and I/O |
| DAC analogue 3.3 V | Texas Instruments **TPS7A20** (TPS7A2033) | Low-noise analogue / clock domain |

That split — cheap regulator for digital, a Texas Instruments ultra-low-noise LDO for analogue — is the difference between a Sabre chip that can still measure like the datasheet and a Sabre chip that measures like a phone dongle.

### The conversion, treated as analogue

The DAC current comes out of the ES9018K2M and meets a discrete I/V and line stage built from five dual audio op-amps (`U6`–`U10`). The gain-setting resistors are 806 Ω, 0.1 %, thin-film 0402 parts — twenty of them — because current-to-voltage is where a Sabre board either becomes transparent or becomes a tone control. The filter capacitors in that path are C0G/NP0 (2.2 nF 2 % and 470 pF), not the X5R that is cheaper to pick.

Five dual packages is not excess. They implement the current-to-voltage conversion and the balanced plus unbalanced line stages, so the single-ended jack is a real analogue output rather than one half of a differential pair left hanging.

### A clock that is not an afterthought

Y1 is a 100 MHz, 3.3 V, 3225 CMOS oscillator, the frequency the ES9018K2M actually wants. The Standard and Pro editions differ here on purpose — see below — but both editions give the converter its own oscillator rather than begging a clock from the Pi.

### A HAT that behaves like a HAT

The board is a mechanical HAT: 65.1 × 56.1 mm, 1.6 mm FR4, 40-pin extended socket, mounting holes, and a cut-out that clears the Pi’s connectors. I²S, I²C, and the usual GPIO pass through. The converter’s I²C address is `0x90` in 8-bit notation, which is **`0x48`** in the 7-bit notation every Linux driver uses.

<p align="center">
  <img src="LightSABRE_DAC.3D-render-02.png" alt="LightSABRE DAC, underside three-quarter view" width="720">
</p>

---

## Standard and Pro

One PCB. Two bills of materials. The difference is not a marketing sticker — it is two parts, chosen because they sit in the two places a listener can still hear after the power architecture is already right.

| | **Standard** — `LightSABRE_DAC.bom.varA.xlsx` | **Pro** — `LightSABRE_DAC.bom.varB.xlsx` |
| --- | --- | --- |
| **Op-amps U6–U10** | Texas Instruments **OPA1678** | Texas Instruments **OPA1602** |
| **Op-amp character** | SoundPlus FET-input, 4.5 nV/√Hz, the honest high-performance default | SoundPlus bipolar-input, 1.1 nV/√Hz, the quieter, more expensive sibling |
| **Oscillator Y1** | CMOS crystal oscillator, 100 MHz / 20 ppm (`O93225100MEDA4SI-13`) | SiTime **SiT8209** MEMS oscillator, 100 MHz / 20 ppm |
| **Clock character** | Correct frequency, serviceable jitter, easy to source | MEMS stability, vibration immunity, the part you specify when the rest of the board deserves it |
| **Everything else** | Identical | Identical |

The isolated DC/DC, the TPS7A20, the 0.1 % I/V resistors, the C0G filter caps, the Sabre chip, the jacks, and the HAT mechanics do not change. Darmur did not ship a “lite” board and a “real” board. He shipped one analogue design and two component grades, so a first build can be affordable and a second build can be uncompromising without a respin.

If you are assembling a single board and the Pro parts are in stock, fit the Pro parts. If you are assembling a small run and want every unit to exist, Standard is not a consolation prize.

---

## What the converter will do

From the ES9018K2M, once the board around it is this careful:

- PCM up to **32-bit / 384 kHz**
- DSD up to **DSD256**
- On-chip ASRC and jitter elimination
- Datasheet territory of **127 dB DNR** and **−120 dB THD+N**
- I²C register map for volume, filters, and de-emphasis

Those figures are the chip’s. The board is designed so they remain plausible after the signal has left the QFN.

---

## Repository contents

Everything required to order, assemble, inspect, and understand Rev 1.0.

| File | What it is |
| --- | --- |
| `LightSABRE_DAC.kicad_pro` / `.kicad_sch` / `.kicad_pcb` | KiCad 9 project. Sheets: **power**, **audio**, **interface**. |
| `power.kicad_sch` | Input protection, isolated ±12 V, ±9 V analogue, split 3.3 V. |
| `audio.kicad_sch` | ES9018K2M, clock, I/V, balanced and unbalanced outputs. |
| `interface.kicad_sch` | Raspberry Pi 40-pin HAT pinout, I²S and I²C. |
| `LightSABRE_DAC.sch.pdf` | Printable schematic. |
| `LightSABRE_DAC.pcb.pdf` | Printable board. |
| `LightSABRE_DAC.bom.varA.xlsx` | **Standard** BOM with MPN, LCSC, Mouser, Digi-Key. |
| `LightSABRE_DAC.bom.varB.xlsx` | **Pro** BOM. Same columns. Different U6–U10 and Y1. |
| `LightSABRE_DAC.pos.csv` | Centroid / pick-and-place. |
| `LightSABRE_DAC.gerber_x1.2-Layers.zip` | 2-layer Gerbers, classic Protel extensions. |
| `LightSABRE_DAC.gerber_x1.4-Layers.zip` | 4-layer Gerbers, classic Protel extensions. |
| `LightSABRE_DAC.gerber_x2.2-Layers.zip` | 2-layer Gerbers, KiCad X2 / `.gbr` naming, split PTH/NPTH drills. |
| `LightSABRE_DAC.gerber_x2.4-Layers.zip` | 4-layer Gerbers, KiCad X2 naming. |
| `LightSABRE_DAC.3D-model.step` | Full 3D for enclosure work. |
| `LightSABRE_DAC.3D-render-0[1-4].png` | The renders on this page. |
| `LICENSE` | CC0 1.0 Universal. |

---

## Make one

The manufacturing pack is not a teaser. It is the order.

### 1. Choose a stack-up

Both 2-layer and 4-layer plots are in the tree. The design rules are 0.1524 mm (6 mil) clearance and trace, 35 µm copper, 1.6 mm finished thickness.

**Order the 4-layer board** unless a fab house forces the issue. Inner planes are how a Sabre HAT keeps digital return current out of the analogue. The 2-layer set exists so the project remains buildable when a four-layer panel is not.

Suggested 4-layer reading of the job file Darmur exported:

| Layer | Role |
| --- | --- |
| F.Cu | Signals, analogue, components |
| In1.Cu | Plane |
| In2.Cu | Plane |
| B.Cu | Signals and HAT socket |

Finish is left open in the job file. ENIG is the grown-up choice for a 0.5 mm-pitch QFN and 0402 I/V network; HASL will work if the assembler is honest about it.

### 2. Choose a Gerber dialect

| Archive prefix | Give this to |
| --- | --- |
| `gerber_x1.*` | Fabs that still want `.gtl` / `.gbl` / `.gto` and a single `.drl` |
| `gerber_x2.*` | Fabs that want modern `.gbr` names and separate PTH / NPTH drills |

If you are unsure, send `LightSABRE_DAC.gerber_x2.4-Layers.zip` and the matching BOM. Most houses in 2026 prefer it.

### 3. Choose Standard or Pro

Upload `LightSABRE_DAC.bom.varA.xlsx` or `LightSABRE_DAC.bom.varB.xlsx` with `LightSABRE_DAC.pos.csv`. LCSC numbers are already in the sheets. Fiducials `FID1`–`FID6` are on the board because this was always meant to go through a pick-and-place.

### 4. Respect the parts that look small

This is not a first hot-air project.

- ES9018K2M is a **5 × 5 mm QFN-28**, 0.5 mm pitch, thermal pad with vias
- Most of the analogue network is **0402**
- The I/V resistors are 0.1 % thin film — do not “equivalent” them to 1 % thick film
- The barrel jack is **5.5 × 2.0 mm**, not the more common 2.1 mm pin
- The isolated DC/DC is a tall SIP. Check enclosure height before you print a case

Professional assembly is the path that honours the layout. A careful hand build is possible; a careless one will spend three evenings chasing a pin that the stencil would have got in eight seconds.

### 5. First power, before first music

1. Inspect the QFN and the DC/DC orientation.
2. Leave `JP1` off.
3. Feed **5.2 V** centre-positive into `J1`. Confirm the input LED.
4. Measure the isolated island: roughly ±12 V after PS1, ±9 V after U1/U2.
5. Measure both 3.3 V rails at the DAC.
6. Only then seat the HAT on a Pi, decide whether `JP1` should be on, and bring I²S up.

If a rail is wrong, stop. The gift is a working board, not a smoked Sabre.

---

## Bring it up on a Raspberry Pi

LightSABRE speaks the language every Pi DAC already speaks.

- **I²S** from the 40-pin header into the ES9018K2M
- **I²C** at **0x48** (7-bit) for volume, filters, and mute
- Overlay starting point on Raspberry Pi OS / Volumio / moOde / DietPi: a generic I²S DAC, then an ES9018K2M userspace or kernel client if you want hardware volume

A community Volumio plugin for ES9018K2M boards lives at [foonerd/es9018k2m-plugin](https://github.com/foonerd/es9018k2m-plugin). Darmur’s own register work on later Sabre parts is the reason that ecosystem is as good as it is. Use what fits; publish what you change.

Suggested first listen, so reports can be compared:

- Same file, same headphones or the same pair of TRS cables
- Unbalanced jack first, then balanced into something that can take it
- One track you know too well, one you have never heard on a Sabre chip
- Note the edition (Standard or Pro), the stack-up (2-layer or 4-layer), the Pi model, and the player

---

## The invitation

Darmur did the expensive part: the thinking, the layout, the variants, the fab files, and the decision to give them away.

The interesting part left is not another respin. It is a board that exists, a listening chair, and a short write-up that the next builder can trust.

Build a Standard. Build a Pro. Build one of each and swap only the op-amps in your notes. Photograph the underside. Post the first-power rail measurements. Say whether the isolated island was worth it on *your* Pi. Open an issue on this fork, or a pull request with a build log, or a thread wherever you already talk about DACs.

If the board taught you something, write it down. That is how an open hardware gift stays alive after the ZIP files stop being news.

---

## Designer

**Dario Murgia ([Darmur](https://github.com/Darmur))**  
Italian electronics engineer. Author of LightSABRE and of a long run of open audio boards — [BassOwl-HAT](https://github.com/Darmur/bassowl-hat), [BassOwl-Lite](https://github.com/Darmur/bassowl-lite), [BassFly-uHAT](https://github.com/Darmur/bassfly-uhat), [BassCrab-uHAT](https://github.com/Darmur/basscrab-uhat), [tubeamp](https://github.com/Darmur/tubeamp). Schematic title block: *itz-embedded*, 2026-01-09, Rev 1.0.

The silkscreen already says who made it. This README exists so a stranger landing on a fork still finds his name first.

---

## License

[CC0 1.0 Universal](LICENSE). To the extent possible under law, the author has waived all copyright and related rights to this work.

ESS SABRE, SABRE32, and related marks are trademarks of ESS Technology, Inc. Raspberry Pi is a trademark of Raspberry Pi Ltd. Texas Instruments and SiTime part names are used here as the actual parts on the BOMs.

---

<p align="center"><em>Make the board. Listen for a week. Tell someone what it did.</em></p>
