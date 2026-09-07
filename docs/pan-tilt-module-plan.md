# CRMX-Controlled Remote Pan/Tilt Mirror Module — Project Plan

## 1. Summary

A wireless-DMX-controlled pan/tilt actuator that aims a CRLS mirror remotely from
a lighting console. The housing has a female 5/8" ("baby") pin receptacle on each
end so it rigs in-line with standard lighting hardware — hang it from a truss
clamp on one end, hang another accessory (or a safety point) off the other.

## 2. Open questions to lock down first

These drive motor sizing, gear ratios, and the mechanical envelope, so they're
the first work item, not an afterthought:

- CRLS mirror dimensions and weight (largest and smallest expected).
- Required pan range (continuous 360° vs. limited sweep) and tilt range.
- Required speed/accuracy (slow smooth sweeps vs. fast "hit-a-mark" moves).
- Environment: indoor-only, touring (drop/vibration), or outdoor (IP rating,
  temperature range).
- Target unit cost and expected production quantity (one-off prototype vs.
  small-batch product changes almost every part choice below).
- Whether genuine CRMX certification/branding is a hard requirement, or a
  generic 2.4 GHz wireless-DMX link (no licensing) is acceptable.

## 3. System architecture

```
Console (DMX) --wireless--> CRMX RX module --wired DMX--> Controller PCB
                                                              |-- Pan motor + driver
                                                              |-- Tilt motor + driver
                                                              |-- Position feedback (optional)
                                                              |-- Power regulation
Housing: female 5/8" baby pin on each end, mirror gimbal/cradle in the middle
```

### 3.1 Wireless link
- Use a certified OEM CRMX RX module (e.g. LumenRadio Nova/SuperNova RXi
  family) rather than reimplementing the CRMX RF protocol. It outputs
  standard DMX512, which is the only interface the rest of the design needs
  to know about. This is the legally supportable path to real CRMX
  interoperability.
- Fallback if certification isn't required: a generic W-DMX/2.4 GHz wireless
  DMX transceiver module — cheaper, no licensing, not literally "CRMX."

### 3.2 Controller
- MCU (e.g. STM32F0/G0) reading DMX512 via an isolated RS-485 transceiver
  (MAX3535/ISO1176-class part), optionally with RDM for remote addressing
  and status.
- Two motor axes (pan, tilt): stepper motors (NEMA 14/17 depending on
  mirror weight) driven by quiet step/dir drivers (e.g. TMC2209) through a
  belt or worm-gear reduction.
- Optional absolute magnetic encoders (e.g. AS5600) per axis for homing and
  closed-loop accuracy — recommended if the mirror needs to return to a
  known position after power cycling.
- Motion profile: trapezoidal/S-curve acceleration to avoid jerky beam
  movement, since any motor vibration shows up directly in the reflected
  beam.

### 3.3 DMX personality (draft — finalize once ranges are known)
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

### 3.4 Mechanical
- Gimbal/cradle sized to the CRLS mirror's mounting points; balance the
  mirror on both axes to minimize motor torque and holding current.
- Housing with a female 5/8" baby pin socket at each end for daisy-chain
  rigging, consistent with standard mirror-ball-motor/lighting-accessory
  convention.
- **Safety**: an overhead rigging device needs a secondary safety
  attachment point and load-rated hardware independent of the baby pins —
  baby pins are a locating/mounting feature, not a certified load path by
  themselves. Plan for a safety cable lug rated for the full assembly
  weight plus margin, per standard entertainment-rigging practice
  (ANSI E1.6 / ESTA guidance).
- IP rating and finish chosen based on indoor/touring/outdoor answer above.

### 3.5 Power
- Locking DC input (or powerCON if run alongside truss power), onboard
  regulation to whatever rails the MCU/drivers/RX module need.

## 4. Development phases

1. **Spec lock** — answer the open questions in §2.
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

## 5. Key risks

- CRMX licensing/certification — see §3.1; don't attempt to reverse-engineer
  the RF protocol.
- Mirror inertia unknown until §2 is answered — could change motor class
  entirely (stepper vs. closed-loop servo).
- Rigging safety is a hard requirement, not a nice-to-have, for anything
  hung overhead.
