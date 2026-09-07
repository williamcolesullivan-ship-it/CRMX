# CRMX-Controlled Remote Pan/Tilt Mirror Module — Project Plan

## 1. Summary

A wireless-DMX-controlled pan/tilt actuator that aims a Godox KNOWLED LiteFlow
reflector panel (used as the "CRLS mirror") remotely from a lighting console.
The housing has a female 5/8" ("baby") pin receptacle on each end: one side
docks to the panel's own rail-mount stud, the other docks to a stand, boom, or
grip arm — matching how these panels already mount.

## 2. CRLS mirror = Godox KNOWLED LiteFlow — confirmed specs

Design target is the full LiteFlow line up to the 20" size. Confirmed sizes
and weights (worst case drives the mechanical/motor design):

| Model       | Size          | Weight            |
|-------------|---------------|-------------------|
| LiteFlow 7  | 3 × 3"        | 46 g (1.6 oz)     |
| LiteFlow 15 | 6 × 6"        | 167 g (5.9 oz)    |
| LiteFlow 25 | 10 × 10"      | 403 g (14.2 oz)   |
| LiteFlow 50 | 20 × 20"      | ~1.43 kg (3.15 lb) — design max |

**Mounting interface (this decides the mechanical approach):** LiteFlow
panels don't have a plain edge — they have a mounting rail on the back, and
Godox sells a "WMS Rail Mount Stud" accessory that clips onto that rail and
terminates in a standard male 5/8" baby pin. So the mirror-side connection on
our module is just a female 5/8" socket accepting that OEM stud — no custom
clamp/bracket needs to be designed or fitted to the panel itself. The
stand-side female socket takes a male stud from whatever the module is rigged
to (C-stand, boom arm, grip head), same as any other grip accessory.

Because the rail mount sits roughly centered on the panel's back, the CG is
close to the pivot point in normal use — gravity torque is modest even at the
1.43 kg worst case. This is a light-duty gimbal, not a heavy touring fixture;
size motors accordingly (see §3.2).

## 3. Spec decisions

- **Speed:** slow, deliberate moves — not a fast "hit-a-mark" fixture. Target
  ~3–10 seconds for a full-range move; the DMX speed channel scales within
  that band rather than allowing snap moves.
- **Range (proposed, pending confirmation):**
  - Pan: ~180–270°. Enough to reposition the reflected beam across a room
    without needing continuous 360° rotation, which would require a slip
    ring this design doesn't otherwise need.
  - Tilt: ~90–120°. Reflection angle changes at 2× the mirror's mechanical
    tilt, so this range covers everything from a steep bounce to a
    near-grazing angle.
  - These are a starting point sized to the use case, not a hard
    requirement — revisit once you've mocked up a few real bounce angles.
- **Environment:** indoors only, both studio and on-location. No IP rating
  needed, but the enclosure should tolerate being handled, packed, and moved
  between sets — not a fragile lab prototype.
- **Quantity:** fewer than 5 units. This is a small hand-built run, not a
  product line — it changes sourcing and process (§4.1, §5) away from
  anything that assumes tooling or OEM minimums.
- **CRMX: required, non-negotiable.** See §4.1 for how this pairs with the
  Blackout app specifically.

## 4. System architecture

```
Console (DMX) --wireless--> CRMX RX module --wired DMX--> Controller PCB
                                                              |-- Pan motor + driver
                                                              |-- Tilt motor + driver
                                                              |-- Position feedback (optional)
                                                              |-- Power regulation
Housing: female 5/8" baby pin on each end, mirror gimbal/cradle in the middle
```

### 4.1 Wireless link — CRMX, and how it pairs with Blackout

CRMX is required, so this section is now load-bearing, not a fallback
discussion.

**The Blackout app itself does not speak CRMX.** Blackout (iPad) outputs
Art-Net/sACN over WiFi/Ethernet. To get CRMX out of that chain you need a
CRMX transmitter/gateway node sitting between the iPad and the fixtures —
e.g. LumenRadio Stardust/Aurora, Exalux Connect One/Connect+, Ratpac AKS+
(all take WiFi Art-Net/sACN in, output CRMX), or a wired USB-C option like
FTSLED Cerise/Cinelex Skycast if you'd rather run a cable to the iPad. That
TX node is **not part of this module** — it's simply the CRMX transmitter
our modules pair with, exactly as they would with any other CRMX fixture.
You've confirmed you already have a CRMX transmitter, so this is resolved —
nothing further to build or buy on the TX side.

**RX side, sized for <5 units:** don't go through LumenRadio's OEM/CRMXchip
channel — that's exactly how Godox and Aputure do it internally (Aputure's
own docs confirm every fixture ships with an embedded LumenRadio CRMX
module), but it means a board-level chip design-in with an NDA and volume
commitment to LumenRadio. Right approach for a manufacturer, wrong one for
five hand-built units.

**Recommended: Godox TimoLink RX ($189, B&H).** This is the accessible
version of the same thing — Godox took the identical LumenRadio CRMX
receiver and put it in a small standalone retail box, at Godox consumer
pricing rather than lighting-industry pricing:
- Built around a genuine LumenRadio CRMX receiver — fully certified, same
  interoperability as any other CRMX product.
- 5V/90mA via USB-C — trivial to power.
- 110×53×26mm, 80g — small enough to mount alongside the gimbal.
- Output: 5-pin female XLR (standard DMX cable) — wire it in via a short
  DMX jumper into an XLR jack on the controller enclosure, rather than
  cutting the connector off, so the unit stays usable standalone.
- Confirmed working with Blackout in the field by outside users.
- Bonus: same brand as the LiteFlow mirrors, so the system stays
  single-vendor.

Alternative: the LumenRadio CRMX Slim RX RDM is IP65-rated with a terminal
block, but at ~$400–600/unit it's 2–3x the cost for ruggedization this
indoor-only build doesn't need. Use it only if a future revision needs
that rating.

### 4.2 Controller
- MCU (e.g. STM32F0/G0) reading DMX512 via an isolated RS-485 transceiver
  (MAX3535/ISO1176-class part), optionally with RDM for remote addressing
  and status.
- Two motor axes (pan, tilt): NEMA 11 or NEMA 14 stepper motors — the 1.43 kg
  worst-case payload with a roughly centered mount point is light duty, so
  NEMA 17 isn't needed and would just add bulk/weight the module doesn't
  need. Drive with quiet step/dir drivers (e.g. TMC2209) through a small
  belt or spur-gear reduction.
- Optional absolute magnetic encoders (e.g. AS5600) per axis for homing and
  closed-loop accuracy — recommended if the panel needs to return to a
  known position after power cycling.
- Motion profile: trapezoidal/S-curve acceleration to avoid jerky beam
  movement, since any motor vibration shows up directly in the reflected
  beam.
- Confirm actual worst-case moment once panel size/mount geometry is
  finalized in CAD (mount rail is centered top-to-bottom on some panels but
  not necessarily left-right — check the datasheet drawing before finalizing
  motor torque).

### 4.3 DMX personality (draft)
| Ch | Function      | Notes |
|----|---------------|-------|
| 1  | Pan (coarse)  | Maps to ~180–270° mechanical range |
| 2  | Pan (fine)    | |
| 3  | Tilt (coarse) | Maps to ~90–120° mechanical range |
| 4  | Tilt (fine)   | |
| 5  | Pan/tilt speed| Scales within the ~3–10s full-range move target — no snap-move mode |
| 6  | Control/macro (home, reset, fine/coarse mode) | |

Build this as a custom fixture profile in Blackout's Fixture Creation Wizard
once the channel map and ranges are final, so it shows up alongside your
other patched fixtures.

### 4.4 Mechanical
- One female 5/8" socket accepts the panel's own Godox WMS rail-mount stud
  directly (no custom bracket for the panel side); the other female 5/8"
  socket accepts a male stud from whatever the module is rigged to
  (C-stand, boom arm, grip head).
- Gimbal sized around the full LiteFlow size range (3" up to 20"); the pivot
  point should land at the panel's rail location so the CG stays close to
  the axes across sizes.
- **Safety**: even at this light payload, anything positioned over a set or
  overhead needs a secondary safety attachment (safety cable/wire) rated for
  the full assembly weight plus margin — standard grip/rigging practice,
  independent of the baby pins (which are a locating/mounting feature, not
  a certified overhead load path by themselves).
- Finish/environment (indoor studio vs. location work) chosen based on the
  open question in §3.

### 4.5 Power
- Small DC barrel or locking connector, on-board regulation to whatever
  rails the MCU/drivers/RX module need. Given the light-duty motors, total
  power draw should be modest — worth checking whether USB-C PD or a small
  battery pack is viable for a fully portable unit, since this is more of a
  grip/location accessory than a truss-mounted stage fixture.

## 5. Development phases

Scoped for a <5-unit hand-built run: 3D-printed mechanical parts (no
tooling/injection molding needed), small-run PCBs (e.g. JLCPCB/OSH Park),
hand assembly. No FCC/CE self-certification needed since these aren't being
sold — the standalone CRMX RX units are already certified, which covers the
only RF component in the design.

1. **Spec lock** — confirm the pan/tilt ranges in §3 once mocked up.
2. **Electrical design** — schematic (KiCad): MCU, DMX input from the
   standalone CRMX RX unit's DMX-out, motor drivers, encoders, power supply.
3. **Mechanical design** — CAD (Fusion 360/SolidWorks): gimbal, motor
   mounts, baby-pin receptacle bosses, cable routing, weight/balance
   analysis across the full LiteFlow size range.
4. **Firmware** — DMX512 receive (+ RDM optional), motion control, homing,
   soft limits, slow acceleration profiles per §3.
5. **Prototype build** — one unit first: dev-board electronics + 3D-printed
   mechanical prototype; validate motion range, DMX responsiveness, and CRMX
   link reliability through your actual Blackout → TX node → RX chain.
6. **Iterate** — refine motor/gear selection from measured behavior; refine
   enclosure for cable strain relief and the rigging safety point.
7. **Testing** — DMX conformance, CRMX range/reliability test in the actual
   studio and location environments you'll use it in, rigging load test.
8. **Build remaining units** — replicate once the first prototype is
   validated.
9. **Documentation** — DMX chart, Blackout fixture profile, wiring diagrams.

## 6. Key risks

- CRMX licensing/certification — see §4.1; don't attempt to reverse-engineer
  the RF protocol; use certified standalone RX units instead of the OEM
  chip channel given the small quantity.
- Verify the rail-mount location on the actual panel (not just the 5/8" stud
  spec) before finalizing motor torque — an off-center rail would raise the
  worst-case moment above the light-duty estimate in §4.2.
- Rigging safety is a hard requirement, not a nice-to-have, for anything
  positioned overhead or over a set, even at these light weights.

## 7. Sources

- [Godox KNOWLED LiteFlow 50 Reflector Kit (20 x 20") — B&H](https://www.bhphotovideo.com/c/product/1790962-REG/godox_liteflow_50_kit_liteflow_50_reflector_kit.html)
- [Godox KNOWLED LiteFlow 50 (NO. 2) — B&H](https://www.bhphotovideo.com/c/product/1790959-REG/godox_liteflow_50_no_2.html)
- [Godox KNOWLED LiteFlow 25 (10 x 10") — Walmart](https://www.walmart.com/ip/Godox-KNOWLED-LiteFlow-25-Soft-Light-Reflector-10-x-10/16312121194)
- [Godox KNOWLED LiteFlow 7 (3 x 3") Reflector Kit — B&H](https://www.bhphotovideo.com/c/product/1790968-REG/godox_liteflow_7_kit_liteflow_7_reflector_kit.html)
- [Godox KNOWLED LiteFlow K1 Reflector Kit — B&H](https://www.bhphotovideo.com/c/product/1790969-REG/godox_liteflow_k1_kit_liteflow_k1_reflector_kit.html)
- [Godox Rail Mount Stud (WMS) for KNOWLED LiteFlow — B&H](https://www.bhphotovideo.com/c/product/1797737-REG/godox_wms_rail_mount_stud_for.html)
- [What are the Godox LiteFlow Cine Light Reflector Panels? — Essential Photo](https://www.essentialphoto.co.uk/blogs/news/what-is-the-godox-liteflow-cine-light-reflector-panel-series)
- [Godox KNOWLED LiteFlow Kits now available — Newsshooter](https://www.newsshooter.com/2024/01/19/godox-knowled-liteflow-kits-now-available/)
- [The Ultimate Guide to Controlling Aputure, ARRI, and Nanlux Lights via CRMX | RTctrl + Blackout App](https://rtctrl.com/solutions/crmx-guide/)
- [Blackout Lighting Console — App Store](https://apps.apple.com/us/app/blackout-lighting-console/id1414562959)
- [Hardware Setup — Blackout User Manual](https://docs.blackout-app.com/manual/introduction/hardware-setup)
- [CRMX OEM Modules | Wireless DMX for Manufacturers — LumenRadio](https://lumenradio.com/wireless-dmx/crmx-oem-modules/)
- [CRMX Slim RX RDM — LumenRadio](https://lumenradio.com/products/crmx-slim-rx-rdm/)
