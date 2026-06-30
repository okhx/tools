# Setting up px://hook for real traffic interception

`px://hook` is a Burp Suite-style request inspector. On its own it can only
see traffic from a single sandboxed browser tab. To capture **real** traffic
from a phone or another device — every method, real headers, decrypted
HTTPS — you need two companion tools, both included in this repo:

- **`sys-proxy`** — a real local MITM proxy. Something has to run this; it's
  the actual interception point.
- **`ca://forge`** — generates the root certificate your device needs to
  trust, entirely in-browser, no install required to *generate* it.

This page is the full setup path for every device combination that's
actually possible today. Read the [device support matrix](#device-support-matrix)
first to find your row, then jump to that section.

---

## Device support matrix

| What you have | What runs the proxy | Possible? |
|---|---|---|
| Windows / Mac / Linux computer | The computer itself | ✅ Yes |
| Android phone | The Android phone itself (via Termux) | ✅ Yes — fully phone-only |
| Android phone + a computer on the same Wi-Fi | The computer | ✅ Yes |
| iPhone + a computer or Android phone on the same Wi-Fi | The computer/Android phone | ✅ Yes |
| **iPhone only, nothing else** | — | ❌ Not possible (see [why](#why-iphone-only-doesnt-work)) |

The short version: **something** needs to run as a real background server
that your device's traffic gets routed through. A computer can always do
this. An Android phone can do this *for itself*, via Termux. An iPhone
cannot do this for itself or anything else — iOS doesn't allow it without a
real native app approved by Apple, which is a different kind of project
than what's in this repo (more on that below, not glossing over it).

---

## How it actually works, briefly

```
Your device ──(proxy setting)──▶ sys-proxy ──▶ real website
                                      │
                          decrypts HTTPS using a cert
                          it mints on the fly, signed
                          by a CA your device trusts
                                      │
                                      ▼
                              px://hook UI
                         (shows every request live)
```

You install one certificate on each device you want to inspect. That's the
only "trust" step. Nothing else about your device changes.

---

## Scenario A — Everything on one computer

The simplest setup. Browse on the same machine running the proxy.

1. **Generate a CA.** Open `okhx.github.io/tools/ca` in any browser. Click **Generate
   CA**. In step 4, download `ca.cert.pem` and `ca.key.pem`, and place both
   inside `sys-proxy/certs/` (create that folder if it doesn't exist).
3. **Install the CA on this computer:**
   - **macOS:** double-click `ca.cert.pem` → opens Keychain Access → find it
     → double-click → expand **Trust** → set "When using this certificate"
     to **Always Trust**.
   - **Windows:** double-click the cert → **Install Certificate** → **Local
     Machine** → **Trusted Root Certification Authorities**.
   - **Linux (Firefox):** `Settings → Privacy & Security → Certificates →
     View Certificates → Authorities → Import`, check "Trust this CA to
     identify websites."
4. **Start the proxy:**
   ```bash
   cd sys-proxy
   npm install
   npm start
   ```
5. **Set your system proxy** to `127.0.0.1:8888` (System Settings → Network
   → Proxies on Mac, Settings → Network & Internet → Proxy on Windows).
6. **Open `okhx.github.io/tools/hook`**, click the gear icon, under **MITM Proxy Link**
   enter `ws://127.0.0.1:8889`, click **Connect**.
7. Browse normally — every request appears live in `px://hook`.

---

## Scenario B — iPhone, with a computer on the same Wi-Fi

1. On the computer, generate a CA (`okhx.github.io/tools/ca`, step 1) and start
   `sys-proxy` (steps 1–2 and 4 from Scenario A) — but skip installing the
   CA on the computer itself; it's the iPhone that needs to trust it.
2. Find the computer's LAN IP:
   - Mac: `ipconfig getifaddr en0`
   - Windows: `ipconfig` (look for IPv4 Address)
   - Linux: `ip addr`
3. **On the iPhone**, open Safari and go to
   `http://<computer-LAN-IP>:8889/ca`. iOS will prompt to download a
   configuration profile — allow it.
4. Go to **Settings → General → VPN & Device Management**, tap the
   downloaded profile, **Install** (passcode required).
5. **This step is easy to miss and required:** go to **Settings → General
   → About → Certificate Trust Settings**, toggle full trust **on** for the
   cert you just installed. Skipping this leaves the profile installed but
   HTTPS will still fail.
6. Go to **Settings → Wi-Fi → (i) next to your network → Configure Proxy →
   Manual**. Server = the computer's LAN IP, Port = `8888`.
7. **On the computer**, open `okhx.github.io/tools/hook`, gear icon, **MITM Proxy Link**,
   enter `ws://127.0.0.1:8889`, **Connect**.
8. Browse on the iPhone — traffic appears live on the computer's screen.

---

## Scenario C — Android phone, fully on its own (no PC needed)

Android can run the entire proxy on itself, using **Termux** (a real Linux
environment for Android, no root required).

1. Install **Termux from F-Droid**: <https://f-droid.org/packages/com.termux/>
   — not the Play Store version, which is outdated and breaks package
   installs.
2. Get this repo's `sys-proxy` folder onto the phone. Easiest path inside
   Termux:
   ```bash
   pkg install git -y
   git clone https://github.com/okhx/tools
   cd sys-proxy
   ```
3. Run the bootstrap script:
   ```bash
   chmod +x termux/setup.sh
   ./termux/setup.sh
   ```
   This installs Node, installs dependencies, generates the CA, and prints
   your phone's own Wi-Fi IP and the exact remaining steps.
4. Start the proxy:
   ```bash
   npm start
   ```
   Or, to keep it running in the background while you use other apps:
   ```bash
   chmod +x termux/keep-alive.sh
   ./termux/keep-alive.sh
   ```
   (Android can still kill background processes on some phones despite
   this — see the note inside that script if the proxy keeps dying.)
5. **Install the CA on this same phone:** open a browser and visit
   `http://127.0.0.1:8889/ca`, then:
   **Settings → Security → Encryption & credentials → Install a
   certificate → CA certificate**, select the downloaded file.
   > Android only trusts user-installed CAs for browser traffic by
   > default (since Android 7+). Most apps will ignore it unless you root
   > the device and install it as a system certificate. Browser-based
   > testing works fine without that extra step.
6. **Set the phone's own Wi-Fi proxy:** long-press your Wi-Fi network →
   **Modify network → Advanced → Proxy → Manual**. Server = the phone's own
   Wi-Fi IP (printed by the setup script — not `127.0.0.1`, Android's proxy
   setting needs a real address even pointing at itself). Port = `8888`.
7. Open `okhx.github.io/tools/hook` in a browser on the phone (or any device that can
   reach the phone's IP) and connect to `ws://<phone-IP>:8889`.

---

## Scenario D — iPhone + Android phone, no computer at all

Combine B and C: run `sys-proxy` on the Android phone (Scenario C, steps
1–4), then follow Scenario B's iPhone steps but point the iPhone's proxy
settings and CA download at the **Android phone's** Wi-Fi IP instead of a
computer's. Both phones need to be on the same Wi-Fi network.

---

## Why iPhone-only doesn't work

This is a deliberate Apple platform restriction, not a gap in this
project. iOS sandboxes every app individually and doesn't allow any app —
including this one, including any web page — to run a background server
process that the system routes other apps' or Safari's traffic through.

There is a real, supported way iOS apps intercept traffic
(`NEAppProxyProvider` / `NEPacketTunnelProvider`, the API every legitimate
iOS packet-capture app uses), but it requires a compiled native Swift app,
a paid Apple Developer account, and Apple's approval for the
network-extension entitlement, distributed through Xcode or the App Store.
That's a fundamentally different kind of project than an HTML page or
script — there's no tutorial that turns a webpage into that, and claiming
otherwise would just waste your time. If you only have an iPhone, you need
one other device (a computer, or even a second Android phone) on the same
network to act as the proxy host.

---

## Troubleshooting

**"is sys-proxy running?" in px://hook** — the proxy isn't started, or
you're connecting to the wrong address. `ws://127.0.0.1:8889` only works
if `px://hook` is open on the *same* machine running `sys-proxy`. If
they're on different devices, use that device's actual LAN/Wi-Fi IP.

**HTTPS sites fail to load after installing the cert** — on iPhone, you
almost certainly skipped the separate **Certificate Trust Settings**
toggle (step 5 in Scenario B) — installing the profile alone isn't enough.

**Some apps still bypass the proxy / fail to connect** — apps using
certificate pinning (banking apps, some messengers) will refuse to trust
any CA except the one they ship with, by design. This is true of Burp and
every other MITM tool too, not specific to this one. Regular browser
traffic isn't affected.

**Termux package installs fail with "Unable to locate package"** — you
installed Termux from the Play Store. Uninstall it and install the F-Droid
build instead; the Play Store version stopped receiving repository updates.

---

## Files in this repo

| File | What it does |
|---|---|
| `px://hook` | Repeater, Intruder, Organizer, vuln scanning, Burp in your pocket |
| `ca.html` | Generates a CA cert entirely in-browser, no install needed to generate it |
| `sys-proxy/` | The actual MITM proxy server (Node) |
| `sys-proxy/termux/setup.sh` | One-shot Android/Termux bootstrap |
| `sys-proxy/termux/keep-alive.sh` | Keeps the proxy alive in the background on Android |