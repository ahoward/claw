# claw

Scripts for running a graphical desktop + browser on `drawohara.dev` (Ubuntu 24.04),
accessible from macOS over Tailscale — no SSH tunneling required.

Also includes setup for [OpenClaw](https://github.com/pjasicek/OpenClaw), an open-source
reimplementation of the 1997 platformer *Captain Claw*.

## How it works

```
Mac (built-in Screen Sharing)  <──tailscale──>  drawohara.dev
                                                  ├── Xvfb :1   (virtual display)
                                                  ├── openbox    (window manager)
                                                  ├── x11vnc     (VNC server on :5900)
                                                  └── Chrome / OpenClaw
```

Tailscale provides the secure private network — VNC is bound to the Tailscale IP only,
not exposed to the internet.

## Setup

### 1. Install dependencies (one-time, run as yourself with sudo)

```bash
./bin/install
```

This installs: `tailscale`, `xvfb`, `x11vnc`, `openbox`, and OpenClaw build deps.

### 2. Authenticate Tailscale (one-time)

```bash
sudo tailscale up
```

Follow the URL it prints to authenticate via the Tailscale admin console.

### 3. Get CLAW.REZ (for OpenClaw)

You need `CLAW.REZ` from the original Captain Claw (1997) game. Place it at:

```
~/claw/CLAW.REZ
```

OpenClaw reads assets from the original game file — it is not redistributed here.

### 4. Start the desktop

```bash
./bin/start
```

This starts Xvfb, openbox, x11vnc, and prints the VNC connection address.

### 5. Connect from macOS

In Finder: **Go → Connect to Server**, enter:

```
vnc://100.x.x.x:5900
```

(replace with your Tailscale IP, shown by `./bin/start` or `tailscale ip -4`)

Or from Terminal:

```bash
open vnc://$(ssh drawohara.dev tailscale ip -4):5900
```

### 6. Stop the desktop

```bash
./bin/stop
```

## Scripts

| Script | Description |
|---|---|
| `bin/install` | Install all dependencies |
| `bin/start` | Start virtual display, window manager, VNC |
| `bin/stop` | Stop all desktop processes |
| `bin/browser` | Launch Chrome on the virtual display |
| `bin/openclaw` | Build and/or launch OpenClaw |

## Notes

- VNC has no password by default (safe because it's Tailscale-only)
- The virtual display is `:1` — override with `DISPLAY_NUM=2 ./bin/start`
- Session survives SSH disconnect
- Re-running `./bin/start` when already running is a no-op
