# Single-Unit Build Plan — CRLS-1

First unit of the eventual <5-unit run. Full sketch and interactive parts
list: https://claude.ai/code/artifact/a28ec21f-591c-4c53-9aaa-8fed5b8aa026

## Total cost: $382.38

| Category | Cost | Share |
|---|---|---|
| Wireless & control (Godox TimoLink RX, control board, MAX485) | $234.00 | 61% |
| Motion (2× NEMA 11, 2× AS5600, belts/pulleys) | $30.00 | 8% |
| Mounting & structure (WMS stud, 2× baby pin receiver, print) | $76.38 | 20% |
| Power & misc | $30.00 | 8% |

The Godox TimoLink RX alone is 49% of the total; the two baby pin
receivers are another 15%. Everything else — control board, motors,
encoders, gears, print material, power, hardware — is the remaining ~36%.

## Parts list

| Part | Qty | Cost | Confidence |
|---|---|---|---|
| Godox TimoLink RX (stripped to bare PCB) | 1 | $189.00 | Confirmed — B&H |
| BTT SKR Mini E3 V3 control board | 1 | $40.00 | Confirmed — retail range $35–70 |
| MAX485 RS-485→TTL breakout | 1 | $5.00 | Confirmed |
| NEMA 11 stepper (pan) | 1 | $12.00 | Estimate |
| NEMA 11 stepper (tilt) | 1 | $12.00 | Estimate |
| AS5600 magnetic encoder breakout | 2 | $6.00 | Confirmed |
| GT2 belt + pulley set | 2 | $12.00 | Estimate |
| Godox WMS rail-mount stud | 1 | $15.00 | Estimate — only seen priced abroad (~HK$80) |
| Baby pin receiver, female 5/8" | 2 | $56.38 | Confirmed — Filmtools, $28.19 ea |
| 3D-printed enclosure & yoke (filament only) | 1 set | $5.00 | Estimate |
| 12V DC power supply | 1 | $10.00 | Estimate |
| Wiring, fasteners, bearings, misc | — | $20.00 | Estimate |
| **Total** | | **$382.38** | |

Only two lines carry real pricing uncertainty (the WMS stud and the basic
NEMA 11 motors) and neither swings the total by more than a few dollars.

## Mockup

The linked artifact has labeled side-view and top-view sketches of the
housing: pan motor + controller stack in the main body, a 3D-printed yoke
carrying the mirror on its own tilt pivot, and a female 5/8" baby pin
socket on each end — one to the stand, one receiving the LiteFlow panel's
own Godox WMS rail stud directly. It's a concept sketch, not dimensioned
for fabrication — see `pan-tilt-module-plan.md` for the mechanical
rationale behind the layout.
