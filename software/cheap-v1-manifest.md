# Cheap V1 hardware manifest (dye-sub photo strips)

**Output:** wedding-booth style glossy **2×6" photo strips** — 2 inches wide, 6 tall, four stacked frames. Two complete strips cut from one 4×6 dye-sub sheet.  
**Payment:** QR → Stripe Checkout. **No reader hardware in V1.**  
**Prices checked:** 2026-07-24 USD, before shipping/tax.

```
+----------- 4" -----------+
|  2x6 strip |  2x6 strip  |
|  photo 1   |  photo 1    |
|  photo 2   |  photo 2    |
|  photo 3   |  photo 3    |
|  photo 4   |  photo 4    |
+--------------------------+
```

---

## Reality check

The dye-sub printer **is** most of the BOM. Floor for real strips ≈ **$800–1,100** before payments/enclosure.

| Printer | Street price | Notes |
|---|---:|---|
| Sinfonia CS2 | **~$675** | Cheapest real strips; auto 2×6 cut |
| DNP DS-RX1HS | **~$695** | Booth workhorse; 700× 4×6/roll |
| DNP DS620A | **~$985–999** | Faster/shallower — best wall fit |

---

## Recommended cheapest-real V1: Sinfonia CS2

| # | Part | Price | Source |
|---|---|---:|---|
| 1 | Sinfonia CS2 dye-sub printer | **$675** | [EventPrinters](https://eventprinters.com/products/sinfonia-cs2-printer) · [Desktop Darkroom](https://desktopdarkroom.com/sinfonia-cs2-compact-photo-printer) |
| 2 | Matching 4×6 media kit | **~$120** | CS2 dealer media |
| 3 | Raspberry Pi 4 Model B 2GB | **$54.99** | [Micro Center](https://www.microcenter.com/product/621439/raspberry-pi-4-model-b-2gb-ddr4) |
| 4 | Camera Module 3 Standard | **$29.25** | [Adafruit #5657](https://www.adafruit.com/product/5657) |
| 5 | Official Pi USB-C 15W PSU | **$8.74** | [Adafruit #4298](https://www.adafruit.com/product/4298) |
| 6 | SanDisk High Endurance 64GB microSD | **$26.99** | [Adorama](https://www.adorama.com/sandisk-high-endurance-64gb-microsdxc-memory-card-uhs-i-v30/p/idshd64gb) |
| 7 | 2× LED arcade buttons + wires | **$9.95** | [Adafruit](https://www.adafruit.com/product/3487) |
| 8 | Waveshare 5″ HDMI (**required** — renders the QR) | **$34.99** | [Waveshare](https://www.waveshare.com/product/displays/lcd-oled/lcd-oled-1/5inch-hdmi-lcd.htm) |

| Config | Parts ≈ |
|---|---:|
| Printer + media | **~$795** |
| + Pi / camera / power / SD / buttons | **~$915** |
| + 5″ screen | **~$950** |
| Landed ballpark | **~$1,000–1,100** |

**Payment adds $0.** The screen doubles as the payment surface — the guest scans a QR and pays on their phone. A Stripe Reader S700 (**$299**) is the planned upgrade once volume justifies it; see `../v1-architecture.md`.

---

## Wall-mount pick: DNP DS620A

| Block | ≈ |
|---|---:|
| DS620A | $984.50–$999 ([PFSNY](https://pfsny.com/dnp-ds620a-dye-sub-printer-for-photo-booths-and-events) · [FotoClub](https://www.fotoclubinc.com/products/DNP-DS620A-Dye-Sub-Photo-Printer__DS620ASET.aspx)) |
| 4×6 media kit | $123.85–$145 ([EventPrinters](https://eventprinters.com/products/dnp-ds620a-4x6-media)) → up to 1,600 strips 2-up |
| Pi + cam + PSU + SD + buttons | ~$130 |
| 5″ screen | $35 |
| **Parts (QR payment)** | **~$1,250–1,290** |
| _+ S700 when upgrading_ | _~$1,550_ |

---

## Capacity pick: DNP DS-RX1HS

- **~$695** ([EventPrinters](https://eventprinters.com/products/dnp-ds-rx1hs-photo-printer))  
- Deeper/heavier bay; fewer media swaps  

---

## Buy order

1. Printer: **CS2** (cheapest) · **RX1HS** (capacity) · **DS620A** (wall)  
2. Matching 4×6 media kit  
3. Pi + Camera Module 3 + buttons + screen  
4. Prove four frames → dual 2×6 → cut strips  
5. QR payment (software only) / enclosure / lights after  

See also: `../printers.md`, `../v1-architecture.md`.
