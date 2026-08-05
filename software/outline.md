# tiny photo booth — software outline

V1 kiosk stack for a wall-mounted unit: Raspberry Pi, **dye-sub photo strip printer** (DNP / Sinfonia), **QR → Stripe Checkout** (no reader hardware), Start / Cancel.

Output: glossy **2×6" strips — 2 inches wide, 6 tall, four stacked frames** — two per 4×6 dye-sub sheet.

Hardware/printer detail: `../v1-architecture.md`, `../printers.md`, `cheap-v1-manifest.md`.

---

## 1. Dev environment

### Recommendation: Docker for logic + mocks; real hardware for print

| Approach | Verdict | Why |
|---|---|---|
| Full Pi emulator (QEMU) | Skip as primary | Slow; no real Camera Module 3 / dye-sub USB fidelity |
| **Docker + mocked devices** | **Day-to-day** | Fast iteration on state machine, Stripe, strip compose, UI |
| Laptop Python venv | Fine early | Compose dual-strip PNGs; pygame UI |
| **Real Pi + real dye-sub** | **Required before ship** | Only place print quality and drivers get proven |

Mock the devices; don’t try to fake a whole booth in QEMU.

### Docker shape

```
┌─────────────────────────────────────────────┐
│  docker compose                             │
│  kiosk-app (Python, mocks) ──▶ backend      │
│       camera / printer / gpio mocks         │
│       backend ──▶ Stripe test mode Checkout │
└─────────────────────────────────────────────┘
```

On device: same app with `HARDWARE=real`. **Stripe secret key stays on the backend**, never on the bar Pi.

---

## 2. System architecture

```
Guest
  │
  ▼
[Start] ──▶ kiosk-app (Pi)
               ├── Camera Module 3 (Picamera2)
               ├── HDMI display (Pygame) ──▶ renders payment QR
               ├── Dye-sub printer USB (DNP / Sinfonia driver or SDK)
               ├── GPIO: Start, Cancel, MOSFETs → LEDs, buzzer
               └── HTTPS ──▶ backend
                               ├── create Checkout Session ($5)
                               ├── return session URL → Pi draws QR
                               └── webhook checkout.session.completed
                                              │
                              guest's phone ──┘  (scans QR, pays)
```

One explicit state machine. Hardware and Stripe are adapters.

---

## 3. Kiosk app

### Responsibilities

1. Idle / countdown / preview / strip preview / fault UI  
2. Debounced buttons; Start disabled until back to IDLE  
3. Printer/media health check **before** charge  
4. Start session → wait PAID  
5. Photo light → countdown → four captures  
6. Compose **color dual 2×6 on 4×6** → save → print → cut  
7. SQLite paid jobs + service reprint  
8. Auto-prune customer photos  
9. systemd `Restart=always` on the app process  

### State machine

```
IDLE → PAYMENT_PENDING → PAID → COUNTDOWN → CAPTURING
    → COMPOSING → PRINTING → COMPLETE → IDLE
Any → FAULT
PAYMENT_PENDING → IDLE  (Cancel before pay only)
```

### Package layout

```
software/
  outline.md
  cheap-v1-manifest.md
  kiosk/                    # (future)
    app/
      main.py
      state_machine.py
      config.py             # mock | pi
      ui/
      camera/
      printer/              # dye-sub driver/SDK + mock (write 4×6 PNG)
      gpio/
      lights.py
      compose.py            # Pillow: dual 2×6 color layout on 4×6
      sessions.py
      stripe_client.py
    tests/
  backend/
  docker-compose.yml
```

### Adapters

| Adapter | Mock | Real |
|---|---|---|
| Camera | Fixtures / webcam | Picamera2 |
| Printer | Write 4×6 PNG to `out/strips/` | DNP/Sinfonia USB print + cut |
| Buttons | Keyboard S/C | gpiozero |
| Lights / buzzer | Log | MOSFET / GPIO |
| Payment | Stripe test mode; auto-complete after N sec | Live Checkout Session + QR render |

---

## 4. Stripe

- Secret key only on backend  
- **Checkout Session per booth session**, tagged `client_reference_id = session_id`  
- Pi renders the returned URL as a QR — never touches card data  
- Test mode first; live keys after reprint/recovery works  

| API | Role |
|---|---|
| `POST /start-session` | Create $5 Checkout Session; return URL to encode as QR |
| `GET /sessions/:id` | `pending` \| `paid` \| `failed` \| `expired` |
| `POST /webhooks/stripe` | `checkout.session.completed` → mark paid |

QR expiry ~3 min, then `expired` → IDLE. Pi long-polls `GET /sessions/:id`.

### S700 migration (later)

Swapping to a physical reader changes only `stripe_client.py` and the backend routes — create a `card_present` PaymentIntent and `POST /v1/terminal/readers/:id/process_payment_intent` instead of a Checkout Session. Same state machine, same `GET /sessions/:id` contract. Keep the payment adapter behind an interface so this is a one-file change.

---

## 5. Strip composition

| Spec | Value |
|---|---|
| Sheet | 4×6" dye-sub (printer native) |
| Layout | Two identical 2×6 strips side by side |
| Frames | Four stacked photos per strip |
| Color | Full color |
| Header/footer | Brand + optional venue line |
| Finish | Printer gloss/matte via driver |

**First milestone:** keyboard → four frames → dual-strip PNG → real dye-sub cut (or mock file).

---

## 6. Reliability checklist

- [ ] Printer/media check before charge  
- [ ] Persist paid job before capture  
- [ ] Keep composed file until print OK; reprint without re-charge  
- [ ] Disable Start until IDLE  
- [ ] Ethernet for the Pi when possible  
- [ ] QR timeout returns to IDLE without stranding the session  
- [ ] Dim perimeter LED while QR is on screen (scan reliability)  
- [ ] Log scan-to-pay conversion — it is the early warning for a dead-signal venue  
- [ ] systemd restart on kiosk process  
- [ ] Service menu: test print, preview, reprint, network, reboot  
- [ ] Auto-prune photos  

---

## 7. Build order

1. Compose + dye-sub print path (bench)  
2. UI + state machine  
3. Backend + Stripe Checkout in test mode + QR render  
4. GPIO / lights / buttons on Pi  
5. Enclosure (size bay to the chosen printer)  
6. S700 later — swap the payment adapter, cut the reserved face band  

---

## 8. Open decisions

| Topic | Lean |
|---|---|
| Print host | Prefer Pi + Linux driver if stable; else small Windows print helper |
| Backend host | Hosted VPS; Pi disposable |
| Printer pick | DS620A wall / CS2 cheapest / RX1HS capacity |
| Venue line on strip | Per-bar config string |

---

## 9. Next coding artifacts

1. `docker-compose.yml` — backend + kiosk mock  
2. State machine + mock adapters  
3. `.env` for Stripe test keys (gitignored)  
4. Golden dual-strip PNG fixture test  
