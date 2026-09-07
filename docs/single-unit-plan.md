# Single-Unit Build Plan — CRLS-1

First unit of the eventual <5-unit run. Full sketch and interactive parts
list: https://claude.ai/code/artifact/a28ec21f-591c-4c53-9aaa-8fed5b8aa026
(pricing table there predates the rev B mechanical redesign below — the
numbers here supersede it)

## Total cost: $370.38

| Category | Cost | Share |
|---|---|---|
| Wireless & control (Godox TimoLink RX, control board, MAX485) | $234.00 | 63% |
| Motion (2× NEMA 11, 2× AS5600) | $30.00 | 8% |
| Mounting & structure (WMS stud, 2× baby pin receiver, print) | $76.38 | 21% |
| Power & misc | $30.00 | 8% |

The Godox TimoLink RX alone is 51% of the total; the two baby pin
receivers are another 15%. Everything else — control board, motors,
encoders, print material, power, hardware — is the remaining ~34%.

## Parts list

| Part | Qty | Cost | Confidence |
|---|---|---|---|
| Godox TimoLink RX (stripped to bare PCB) | 1 | $189.00 | Confirmed — B&H |
| BTT SKR Mini E3 V3 control board | 1 | $40.00 | Confirmed — retail range $35–70 |
| MAX485 RS-485→TTL breakout | 1 | $5.00 | Confirmed |
| NEMA 11 stepper (pan) | 1 | $12.00 | Estimate |
| NEMA 11 stepper (tilt) | 1 | $12.00 | Estimate |
| AS5600 magnetic encoder breakout | 2 | $6.00 | Confirmed |
| Godox WMS rail-mount stud | 1 | $15.00 | Estimate — only seen priced abroad (~HK$80) |
| Baby pin receiver, female 5/8" | 2 | $56.38 | Confirmed — Filmtools, $28.19 ea |
| 3D-printed housing (filament only) | 1 set | $5.00 | Estimate |
| 12V DC power supply | 1 | $10.00 | Estimate |
| Wiring, fasteners, bearings, misc | — | $20.00 | Estimate |
| **Total** | | **$370.38** | |

Down $12 from the first pass: rev B (below) drives both motors direct,
so the belt/pulley reduction that rev A needed is gone entirely — not
just cheaper, but one less thing to tension, wear, or backlash.

Only two lines still carry real pricing uncertainty (the WMS stud and the
basic NEMA 11 motors) and neither swings the total by more than a few
dollars.

## Mechanical design — rev B

The first pass (linked sketch above) used a two-arm yoke borrowed from
stage moving-head fixtures, reaching down from a housing near the stand
mount to grab the mirror at a remote pivot. That's the wrong template for
this hardware: the LiteFlow panel is thin and light with a single
center-back mount point already built in (the WMS rail stud), not the
heavy centered lamp engine a yoke is meant to cradle. It also forced the
tilt motor ~10+ inches from the pan motor, needing a belt drive to reach it.

**Current design:** one compact housing sandwiches directly between the
two baby pin sockets — stand-side socket on the back, both motors
co-located and direct-drive inside, a short standoff stud on the front
connecting to the mirror's own WMS stud. No arms, no belt.

The real trade-off: the standoff has to be long enough that the panel's
edge clears the housing as it tilts, and that requirement scales with
panel size. At a 5" standoff, the 3" and 10" panels clear the full tilt
range with room to spare; the 20" panel starts crowding the housing past
roughly ±24° of tilt. Fixing that for the full range at 20" means either a
longer standoff or a notch in the housing — a real design decision, not
a blocker.

## 3D model

Interactive model, rev B, at true relative scale (1 unit = 1 inch):
https://claude.ai/code/artifact/e48f5b2e-a5e3-4563-9f12-0844e0b88291

Drag to orbit, scroll to zoom. Pan and tilt sliders drive the actual
proposed ranges (~180–270° pan, ~90–120° tilt); a size selector swaps
between the 3", 10", and 20" LiteFlow panels on the same fixed standoff;
an internals toggle reveals both motors and the controller stack inside
the one housing; and a live warning appears whenever the current size/tilt
combination would run the panel into the housing, naming exactly how much
clearance is left. Not dimensioned for fabrication — see
`pan-tilt-module-plan.md` §4.4 for the full mechanical rationale.
