# tiny photo booth — V1 architecture

Wall-mounted dive-bar photo booth. Output is **glossy 2×6" dye-sub photo strips** (wedding-booth format).

---

## Product loop

Idle → Start → **QR on screen → guest pays $5 on their phone** → countdown → four photos → compose dual 2×6 layout → dye-sub print → cutter splits into two strips → idle.

```
Camera ─┐
Screen ─┤                                    (QR renders here)
Buttons ├── Computer ── USB ── Dye-sub photo printer (DNP / Sinfonia)
Lights ─┘      │
               └── HTTPS ── backend ── Stripe Checkout ── guest's phone
```

**No payment hardware in V1.** A physical Stripe Reader S700 is the planned upgrade — see *Payment* below.

---

## Printer (non-negotiable)

Roll-fed **dye-sublimation** only. Software lays out two identical strips on one 4×6; the printer cuts them apart.

| Choice | Role |
|---|---|
| **DNP DS620A** | Preferred for wall mount — faster, shallower |
| **DNP DS-RX1HS** | Cheaper DNP / higher roll capacity |
| **Sinfonia CS2** | Cheapest legitimate strip printer |

Details and prices: `software/cheap-v1-manifest.md` and `printers.md`.

---

## Concrete hardware

| Component | V1 choice | Reason |
|---|---|---|
| Main computer | Raspberry Pi 5 (4 GB) or Pi 4 (2–4 GB) | Preview, compose, UI; print via USB/driver |
| Camera | Raspberry Pi Camera Module 3 standard | 12 MP, autofocus, Picamera2 |
| Display | 5–7″ non-touch HDMI | Preview, countdown, **payment QR** |
| Printer | DNP DS620A (or CS2 / RX1HS) | Real 2×6 glossy strips |
| Media | Matching 4×6 dye-sub paper + ribbon kit | 2-up → two strips per sheet |
| Payment | **None — QR to Stripe Checkout** | Guest pays on their own phone |
| Controls | Illuminated Start + Cancel | Minimal UI |
| Photo lighting | Diffused 12 V LED ring or bars at camera | Perimeter glow alone is not enough |
| Decorative lighting | 12 V COB strip behind translucent bezel | Match mockup |
| Switching | Two-channel logic-level MOSFET board | Pi controls photo + decorative lights |
| Sound | Small 5 V active buzzer | Countdown / confirm |
| Storage | 32–64 GB high-endurance microSD | OS, temp photos, failed jobs |
| Power | Separate supplies for Pi, printer, 12 V LEDs | No custom mains inside shell |

---

## Strip layout

```
┌─────────── 4 inches ───────────┐
│  2×6 strip   │   2×6 strip     │
│  photo 1     │   photo 1       │
│  photo 2     │   photo 2       │
│  photo 3     │   photo 3       │
│  photo 4     │   photo 4       │
└────────────────────────────────┘
```

Compose full-color frames. Print at the printer’s native 4×6 / 2-up strip mode. Physical result: two laminated ~2×6" keepsakes per session (or hand both to the guest).

---

## Payment

**V1: QR code → Stripe Checkout.** No payment hardware. The booth renders a QR on its own screen; the guest scans it and pays on their phone.

- Backend creates a Checkout Session per booth session, tagged with `client_reference_id`
- Pi renders the session URL as a QR — **≥ 2″ square** on screen
- `checkout.session.completed` webhook → backend marks paid → Pi advances
- Pi never holds the Stripe secret key
- QR expires after ~3 min → back to IDLE
- US online fees ~2.9% + 30¢ (**~45¢ on $5**)

Flow:

1. Idle: sample strips + “$5 — Press Start”  
2. Start → printer online / media OK  
3. Screen shows QR + “Scan to pay $5”; perimeter LED dims so the screen reads cleanly  
4. Paid → photo light → 3s countdown  
5. Four shots ~1.5s apart → show strip preview  
6. Print + cut → idle  

Cancel any time before payment succeeds; QR also times out on its own.

Hidden service button (~3s hold): printer test, camera preview, reprint last paid job, network status, reboot.

### Going forward: Stripe Reader S700

A physical reader is the expected V2. Reasons to make the jump:

- **Fees:** card-present is ~2.7% + 5¢ (~18¢ on $5) versus ~45¢ for QR. The 26¢/session gap pays off a $299 S700 in ~1,150 sessions.
- **Signal:** QR depends on the *guest's* phone having data inside the bar. A basement room with dead coverage takes zero money no matter how good the booth's own network is. Survey venues for signal before committing to QR-only.
- **Conversion:** a 3-second tap beats 30 seconds of scanning and typing at closing time.

Integration is **server-driven Terminal** — plain REST from the Pi's backend, no mobile SDK. Reserve the ~3″ face band and a spare power lead now (see `casing-study/`).

### Ruled out

- **Cheap USB magstripe swiper** — puts raw card data in our environment, which forces PCI **SAQ D** (~329 controls, paid assessor, quarterly scans). Non-starter.
- **Stripe Reader M2 / mobile Bluetooth readers** — require the iOS/Android SDK. A Pi cannot drive them.
- **Square Terminal** — Square's own guidance says the Terminal API is not for unattended kiosks; an employee must be in view of the device.
- **Vending readers (Nayax, Cantaloupe)** — genuinely simple (they fire a GPIO pulse and need zero payment code), but $259–499 hardware plus ~$10/mo and 5.95% per transaction. Strictly worse economics than Stripe here.

---

## Software stack

| Job | Software |
|---|---|
| Camera | Picamera2 |
| Compose | Pillow (color dual-strip 4×6) |
| UI | Pygame fullscreen |
| Buttons | gpiozero |
| Print | DNP/Sinfonia driver or SDK (not ESC/POS) |
| Payments | Stripe + small HTTPS backend |
| State | SQLite |
| Process | systemd `Restart=always` |

State machine: `IDLE → PAYMENT_PENDING → PAID → COUNTDOWN → CAPTURING → COMPOSING → PRINTING → COMPLETE → IDLE` (+ `FAULT`).

---

## Electronics (brief)

- Buttons: GPIO to GND, pull-ups, software debounce  
- Lights: MOSFET-switched 12 V — never from GPIO power pins  
- Photo light on ~0.5s before each exposure  

---

## Enclosure

Dye-sub printers are deep. Size the bay for the chosen model (DS620A ~10.8 × 14.4 × 6.7"; RX1HS larger).

- Structural rear plate to studs; cosmetic shell does not carry load  
- Printer on rails/tray for media changes  
- Removable chute / exit for the strip  
- PETG/ASA, heat-set inserts, tamper-resistant screws  

Split parts: rear chassis, camera/light pod, screen/button bezel, printer bay, reader cradle, outer shell.

---

## Budget (order of magnitude)

| Area | Approx |
|---|---|
| Dye-sub printer | $675–1,000 |
| Media (first kit) | $120–145 |
| Pi, camera, PSU, storage | $100–160 |
| Display, buttons, LEDs, wiring | $80–150 |
| Payment hardware | **$0 — QR** |
| Shell / plate | $60–150 |
| **Deployed (QR payment)** | **~$1,050–1,600** |
| **Bench (minimal shell)** | **~$900–1,200** |
| _+ S700 when upgrading_ | _+$299 → ~$1,350–1,900_ |

See `software/cheap-v1-manifest.md` for sourced line items.

---

## Build order

1. Camera → compose dual 2×6 → print on real dye-sub (bench)  
2. Display + physical buttons  
3. Stripe Checkout in test mode → QR on screen → session persistence / reprint  
4. Lights + enclosure  
5. S700 later, when fee savings or venue signal problems justify it  

Core V1: **Pi + Camera Module 3 + simple screen + two buttons + dye-sub strip printer + LED circuits.** Payment is software only.
