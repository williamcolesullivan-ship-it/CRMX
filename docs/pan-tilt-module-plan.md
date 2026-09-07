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

## 3. Remaining open questions

These still need answers before locking the design, though they're smaller
in scope now that the payload is known:

- Required pan range (continuous 360° vs. limited sweep) and tilt range —
  driven by how the panel will be used to redirect a beam.
- Required speed/accuracy (slow smooth sweeps vs. fast "hit-a-mark" moves).
- Environment: studio-only, or does it need to survive being knocked around
  a set/location (drop/vibration), or outdoors (IP rating, temp range)?
- Target unit cost and expected production quantity (one-off vs. small
  batch changes part choices below).
- Whether genuine CRMX certification/branding is a hard requirement, or a
  generic 2.4 GHz wireless-DMX link (no licensing) is acceptable.

## 4. System architecture

```
Console (DMX) --wireless--> CRMX RX module --wired DMX--> Controller PCB
                                                              |-- Pan motor + driver
                                                              |-- Tilt motor + driver
                                                              |-- Position feedback (optional)
                                                              |-- Power regulation
Housing: female 5/8" baby pin on each end, mirror gimbal/cradle in the middle
```

### 4.1 Wireless link
- Use a certified OEM CRMX RX module (e.g. LumenRadio Nova/SuperNova RXi
  family) rather than reimplementing the CRMX RF protocol. It outputs
  standard DMX512, which is the only interface the rest of the design needs
  to know about. This is the legally supportable path to real CRMX
  interoperability.
- Fallback if certification isn't required: a generic W-DMX/2.4 GHz wireless
  DMX transceiver module — cheaper, no licensing, not literally "CRMX."

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

### 4.3 DMX personality (draft — finalize once ranges are known)
| Ch | Function      |
|----|---------------|
| 1  | Pan (coarse)  |
| 2  | Pan (fine)    |
| 3  | Tilt (coarse) |
| 4  | Tilt (fine)   |
| 5  | Pan/tilt speed|
| 6  | Control/macro (home, reset, fine/coarse mode) |

Publish this as a standard fixture profile (GDTF/.xml or vendor-specific
format) once locked, so it can be imported into consoles.

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

1. **Spec lock** — answer the remaining open questions in §3.
2. **Electrical design** — schematic (KiCad): MCU, DMX/RDM interface, CRMX
   RX module integration, motor drivers, encoders, power supply.
3. **Mechanical design** — CAD (Fusion 360/SolidWorks): gimbal, motor
   mounts, baby-pin receptacle bosses, cable routing, weight/balance
   analysis for the mirror payload.
4. **Firmware** — DMX512 receive (+ RDM optional), motion control, homing,
   soft limits, acceleration profiles.
5. **Prototype build** — dev-board electronics + 3D-printed mechanical
   prototype; validate motion range, DMX responsiveness, wireless range and
   reliability.
6. **Iterate** — refine motor/gear selection from measured mirror inertia;
   refine enclosure for cable strain relief and the rigging safety point.
7. **Testing** — DMX conformance, 2.4 GHz coexistence (interference from
   Wi-Fi/other wireless DMX gear), thermal, vibration/drop (if touring),
   rigging load test.
8. **Documentation** — DMX chart/fixture profile, wiring diagrams, user
   manual.
9. **Compliance (if going beyond a one-off)** — FCC/CE for the wireless
   module (usually pre-certified if bought as an OEM module), UL/ETL for
   the electrical assembly if sold commercially.

## 6. Key risks

- CRMX licensing/certification — see §4.1; don't attempt to reverse-engineer
  the RF protocol.
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
