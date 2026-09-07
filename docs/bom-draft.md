# Draft Bill of Materials — CRMX Pan/Tilt LiteFlow Module

Sized for a <5-unit hand-built run (per §5 of the project plan). Two
electrical approaches are worth weighing before committing to one.

## Electrical approach: two options

**Option A — reuse a 3D-printer control board.** Boards like the BTT SKR
Mini E3 (STM32F103-based) already integrate an MCU, 2+ TMC2209 stepper
driver sockets, and USB, for well under $30. Add a small MAX485-based DMX
input breakout on a spare UART and flash custom firmware. This skips
custom PCB design/fab entirely — a big time savings at this quantity,
since you're not amortizing NRE cost over a production run anyway.

**Option B — custom PCB.** Cleaner integration (everything on one board,
sized to the enclosure), but requires schematic capture, a fab run
(JLCPCB/OSH Park), and hand assembly per unit. Worth it if the 3D-printer
board's form factor doesn't fit the housing, or you want a more finished
result.

**Recommendation:** start with Option A for the first prototype — it gets
you to a working motion/DMX test fastest — then move to Option B only if
the form factor or finish demands it.

## Per-unit BOM (Option A path)

| Item | Example part | Qty/unit | Notes |
|---|---|---|---|
| CRMX receiver | Godox TimoLink RX, **stripped to bare PCB** | 1 | **$189 (B&H)** for the stock unit. Built around a genuine LumenRadio CRMX receiver, same as Godox/Aputure use internally, just packaged as a retail unit. Desolder the XLR and USB-C connectors and wire directly to the exposed pads — see §4.1 "Chosen integration" in the plan for the procedure. One-way mod: no warranty, no standalone reuse, no public teardown to reference — validate the stock unit works with your transmitter/Blackout before cutting it open. |
| Control board | BTT SKR Mini E3 V3 (or similar STM32-based 3D-printer board) | 1 | ~$25–30. Integrates MCU + 2 TMC2209 driver sockets. |
| DMX input | MAX485-based RS-485-to-TTL breakout, wired directly to the TimoLink RX's exposed DMX pads (no XLR jack/cable in the final build) | 1 | ~$3–5. Feeds a spare UART on the control board. Skip this stage entirely if the TimoLink RX turns out to expose the CRMX chip's TTL-level DMX interface on accessible test points (worth checking during teardown). |
| Pan motor | NEMA 11 or NEMA 14 stepper | 1 | ~$10–15. Sized for the light LiteFlow payload (§4.2 of the plan) — don't over-spec to NEMA 17. |
| Tilt motor | NEMA 11 or NEMA 14 stepper | 1 | ~$10–15. |
| Position feedback | AS5600 magnetic encoder breakout | 2 | ~$5 each. One per axis, for homing/closed-loop accuracy. |
| Belt/pulley or spur gear set | GT2 belt + pulleys, or small spur gears | 2 sets | ~$5–10/set. Reduction for each axis. |
| Mirror-side mount hardware | Godox WMS Rail Mount Stud | 1 | Confirmed OEM accessory — clips to the LiteFlow panel's back rail, outputs a male 5/8" baby pin. |
| Baby pin receiver (both housing ends) | Standard grip "baby pin receiver" hardware (e.g. Filmtools baby pin receiver, female 5/8" socket with threaded mounting base) | 2 | Off-the-shelf grip part — bolts into the 3D-printed housing rather than being custom-machined. |
| Enclosure | 3D-printed (PETG or nylon for durability) | 1 set | Printed in-house or via a print service; fine at this quantity. |
| Power supply | 12V or 24V DC (matched to the control board/driver choice), locking barrel or similar | 1 | Check current draw once motors are selected; light-duty motors should keep this modest. |
| Misc hardware | Bearings, fasteners, wiring, connectors | — | |

## CRMX receiver: how the pick changed

Companies like Godox and Aputure don't buy a boxed receiver at all — they
license LumenRadio's board-level CRMX module ("CRMXchip"/OE-GRX1) and solder
it onto their own fixture PCB. That requires a design-in relationship with
LumenRadio (dev kit, NDA, volume commitment) — the right call at
manufacturing scale, the wrong one for 5 hand-built units.

The LumenRadio **CRMX Slim RX RDM** (the original pick) is a standalone box
around that same chip, but priced for the lighting-rental industry:

| Source | Price |
|---|---|
| Deejay-House (Germany) | €535.77 (~$580 USD) |
| eBay UK listing | $400 USD (likely discounted/used stock) |
| KEL-PLS (New Zealand) | NZD $1,400 ex. GST (~$825 USD, import pricing) |

The **Godox TimoLink RX** is the same idea — a standalone box around a
genuine LumenRadio CRMX receiver — but priced for the consumer/prosumer
photo-video market instead: **$189 at B&H**, a real listed price, no dealer
quote needed. That's the pick now used in the BOM above, saving roughly
$1,000–2,000 across 5 units with no loss of certification or
interoperability (it's the same underlying CRMX technology).

Keep the Slim RX RDM in mind only if a future revision needs its IP65
rating or terminal-block wiring — not relevant for this indoor-only build.

## Cost caveat

Other prices above are rough market ranges from general knowledge of these
parts, not live quotes. They're common maker-community hardware with
well-known street pricing, but confirm current prices before finalizing a
per-unit cost.

## Next steps

1. Order one Godox TimoLink RX and confirm it pairs cleanly with your
   existing CRMX transmitter and Blackout setup before buying 4 more.
2. Confirm the control-board choice by checking its GPIO/UART count against
   the DMX input + 2 stepper drivers + 2 encoders (I2C, so they can share a
   bus) requirement.
3. Order one set of parts and build the first prototype per §5 phase 5 of
   the project plan.
