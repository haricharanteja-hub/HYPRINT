# 🖨️ HYPRINT — Autonomous Campus Cloud Printing Kiosk

> **Self-Service, Autonomous University Printing Under 60 Seconds**  
> No pen drives. No WhatsApp Web. No external shopkeepers. Zero PINs.

---

## 📖 1. Overview & Mission

**HYPRINT** is an enterprise-grade, self-service cloud printing kiosk designed for college campuses and university libraries. Students walk up to the physical kiosk, scan a dynamic QR code on their smartphone, configure their documents (page range, N-up layouts, copies, graph paper), pay seamlessly via UPI / Razorpay, and receive continuous physical printouts in seconds.

The system is engineered for zero-trust environments with hardware telemetry, real-time sensor monitoring, sub-second WebSocket progress tracking, and complete privacy (ephemeral RAM-backed document caching with automated file destruction).

---

## 🌐 2. The 3 Web Interfaces

HYPRINT contains three distinct web interfaces designed for students, the kiosk ambient terminal, and administrators:

```
┌────────────────────────────────────────────────────────────────────────┐
│                          HYPRINT ECOSYSTEM                             │
├─────────────────────────┬───────────────────────┬──────────────────────┤
│    1. Student Portal    │   2. Kiosk Terminal   │  3. Admin Dashboard  │
│      (`index.html`)     │     (`kiosk.html`)    │     (`admin.html`)   │
├─────────────────────────┼───────────────────────┼──────────────────────┤
│ • Mobile-first UI       │ • Ambient kiosk UI    │ • Hardware telemetry │
│ • Multi-format uploads  │ • Dynamic QR generator│ • Real-time sensors  │
│ • 1:√2 A4 page preview  │ • Live queue monitor  │ • Spooler management │
│ • 1, 2, 4, 6 N-up       │ • Sound effects       │ • Paper tray refill  │
│ • Custom range parser   │ • Maintenance alerts  │ • Financial analytics│
│ • Razorpay UPI checkout │ • Fullscreen lock     │ • Remote overrides   │
└─────────────────────────┴───────────────────────┴──────────────────────┘
```

### 📱 1. Student Web App (`public/index.html`)
- **Access:** Opened by scanning the kiosk QR code (`https://<domain>/?kiosk_id=0004`).
- **Universal Ingestion:** Accepts PDF, Word (.docx), PowerPoint (.pptx), Excel (.xlsx), Text (.txt, .rtf, .odt), and Images (PNG, JPG, WebP, TIFF).
- **Pixel-Accurate 1:√2 A4 Previews:** Server-rendered previews preserving exact ISO 216 aspect ratios. Scanned and wide documents automatically select proper orientation.
- **N-up Sheet Imposition:** Imposes 1, 2, 4, or 6 pages onto physical A4 sheets with calculated margins and gutters.
- **Strict Page Range Engine:** Validates page selections (e.g. `1-3, 5, 8-10`) with live pricing updates (`doc_sheets = ceil(pages / nup)`).
- **Engineering Graph Paper:** Append 1:1 millimetre grid sheets directly into the cart without separate document uploads.
- **Zero-PIN Autonomous Release:** Eliminates archaic release codes. Payment verification directly triggers physical print spooling.

### 🖥️ 2. Kiosk Ambient Terminal (`public/kiosk.html`)
- **Access:** Displayed on the physical kiosk PC or touch monitor (`https://<domain>/kiosk`).
- **Dynamic QR Code:** Auto-generates a high-contrast QR code pointing to the live student portal with the kiosk identifier.
- **Live Queue Animations:** Sub-second WebSocket animation displaying active jobs, processing states, and printing progress.
- **Hardware Beacon:** Displays live HP LaserJet status, paper inventory gauge, and friendly maintenance banners if attention is needed.

### 🛡️ 3. Admin & Telemetry Dashboard (`public/admin.html`)
- **Access:** Dedicated management dashboard (`https://<domain>/admin`).
- **Telemetry Sensors:** Real-time diagnostics for Paper Jam, Door/Cover Open, Toner Low, Offline, and Hardware Error bits.
- **Paper Inventory Management:** Tracks remaining sheets in the tray, logs refills, and warns before the tray runs empty.
- **Print Queue Control:** Instant queue purging, job retry, and emergency pause.
- **Financial Analytics & Refunds:** Transaction history, revenue breakdown, and one-click Razorpay refund overrides.

### 📄 Supporting Static Pages
- `contact.html` — Campus support contact and grievance details.
- `shipping.html` — Instant on-kiosk fulfillment policy.
- `refund.html` — Automated transaction and failed print refund policy.
- `privacy.html` — Data privacy declaration (10-minute auto-destruction).
- `terms.html` — Service terms and campus acceptable use policy.

---

## ⚙️ 3. Physical Hardware Architecture

| Component | Hardware Specification | Configuration / Driver |
| :--- | :--- | :--- |
| **Laser Printer** | **HP LaserJet Pro P1108 / P1100 Series** | Simplex Desktop Monochrome Laser, HP 88A (CC388A) Cartridge, Port `USB001` |
| **Print Engine** | **Fast 1-Bit Streaming GDI Engine** | 600 DPI pre-dithered monochrome bitmaps (~4 MB/page), pipelined rendering (~0.36s/page) to feed the P1108's 2 MB hardware buffer continuously without pausing |
| **Status Beacon** | **Arduino Uno R3** | Port `COM9` @ 9600 baud. Digital Pin 13 / built-in "L" LED pulses **3 times (1-sec gap)** upon successful student upload |
| **Host PC** | **Windows 10/11 x64 Kiosk PC** | Connected to Printer (USB001) and Arduino Uno (COM9) |

---

## 🛠️ 4. Tech Stack

- **Backend:** Python 3.11, [FastAPI](https://fastapi.tiangolo.com/), Uvicorn, SQLAlchemy, SQLite (`print_kiosk.db`).
- **Document Processing:** [PyMuPDF](https://pymupdf.readthedocs.io/) (`fitz`), Pillow, PyPDF.
- **Windows Subsystem:** Native Windows Spooler GDI via `pywin32` (`win32print`, `win32gui`, `win32process`), WMI / CIM.
- **Hardware Serial:** `pyserial` (thread-safe asynchronous queue manager).
- **Public Tunnel:** Cloudflare Quick Tunnels (`cloudflared`).
- **Cloud Hosting:** Google Firebase Hosting (global edge CDN).
- **Payments:** Razorpay Payments API (UPI QR, Cards, Netbanking) with server-side HMAC-SHA256 signature verification.

---

## 🚀 5. Getting Started & Setup

### Prerequisites
- Windows 10/11 PC connected to HP LaserJet P1108 via USB.
- Python 3.11+ installed and added to `PATH`.
- Node.js & Firebase CLI (`npm install -g firebase-tools`).
- Arduino IDE (for initial sketch flashing only).

### Step 1: Clone Repository
```bash
git clone https://github.com/your-username/hyprint-kiosk.git
cd hyprint-kiosk
```

### Step 2: Set Up Virtual Environment & Dependencies
```bash
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
```

### Step 3: Configure Environment Variables
Copy `.env.example` to `.env`:
```bash
copy .env.example .env
```
Open `.env` and fill in your keys:
```env
KIOSK_ID=0004
KIOSK_PRINTER_NAME=HP LaserJet Pro P1100 plus series
KIOSK_LED_PORT=COM9
RAZORPAY_KEY_ID=rzp_live_your_key_id
RAZORPAY_KEY_SECRET=your_secret_key
KIOSK_PUBLIC_BASE_URL=https://your-app.web.app
```

### Step 4: Flash the Arduino Uno (Upload Indicator)
1. Open `arduino/hyprint_led/hyprint_led.ino` in Arduino IDE.
2. Select **Board: Arduino Uno**, **Port: COM9**.
3. Click **Upload (→)**.
4. Close Arduino IDE so `COM9` is available for the kiosk backend.

### Step 5: Start the Kiosk System
Run the automated supervisor:
```bash
python auto_tunnel.py
```
This supervisor will:
1. Start the FastAPI backend on `http://127.0.0.1:8000`.
2. Connect to the Arduino Uno on `COM9`.
3. Open a secure Cloudflare Tunnel (`https://<random>.trycloudflare.com`).
4. Update `api_config.json` with the tunnel URL.
5. Automatically deploy the frontend to Firebase Hosting (`hyprint-64865.web.app`).
6. Monitor printer health and restart any degraded services.

---

## 📁 6. Repository File Structure

```
HYPRINT/
├── auto_tunnel.py           # Watchdog supervisor (Uvicorn + Cloudflare + Firebase)
├── main.py                  # Core FastAPI backend, routes, WebSockets & orders
├── file_processor.py        # Strict range parser, A4 1:√2 geometry & N-up imposition
├── cups_manager.py          # Fast 1-bit streaming GDI print engine & spooler monitor
├── hardware_monitor.py      # Low-level Windows spooler & USB telemetry watcher
├── led_notifier.py          # Asynchronous serial worker for Arduino Uno (COM9)
├── config.py                # Environment configurations & defaults
├── database.py              # SQLite models (Jobs, Transactions, Hardware Logs)
├── requirements.txt         # Production Python dependencies
├── firebase.json            # Firebase Hosting configuration & rewrites
├── .env.example             # Safe template for environment variables
├── .gitignore               # Strict git ignore protecting keys and databases
├── README.md                # Master documentation
│
├── arduino/
│   └── hyprint_led/
│       └── hyprint_led.ino  # Arduino Uno pin 13 upload blink sketch
│
└── public/                  # Static frontend assets (Deployed to Firebase)
    ├── index.html           # 1. Student mobile/desktop upload & checkout portal
    ├── kiosk.html           # 2. Ambient kiosk terminal display & QR terminal
    ├── admin.html           # 3. Staff administrative telemetry dashboard
    ├── contact.html         # Contact & support policy
    ├── shipping.html        # Instant kiosk delivery policy
    ├── refund.html          # Automated refund policy
    ├── privacy.html         # Privacy & data retention policy
    ├── terms.html           # Terms and conditions
    ├── api_config.json      # Dynamic pointer to live Cloudflare tunnel
    └── graph_paper.pdf      # High-precision 2mm A4 engineering grid template
```

---

## 🔒 7. Privacy & Security

- **Ephemeral Document Storage:** Finished documents (printed, failed, or refunded) are destroyed **10 minutes** after completion.
- **Unpaid Expiry:** Unpaid jobs are automatically purged after **30 minutes**.
- **HMAC Payment Validation:** Razorpay signatures are strictly validated on the server side; order specifications (copies, pages) are frozen at checkout creation to prevent tampering.
- **Local-Only Admin:** Administrative routes require either a genuine localhost connection on the kiosk machine or an authenticated `KIOSK_ADMIN_TOKEN`.

---

## 📄 License
This project is proprietary and developed for campus cloud kiosk deployments. All rights reserved.
