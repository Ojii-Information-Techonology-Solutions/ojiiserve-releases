# Ojii Serve

A self-hosted LibSQL database manager with a web dashboard, Cloudflare Tunnel integration, and passkey authentication.

Run it on any machine — your Mac, an old PC, a Raspberry Pi, or an Android phone via Termux. Manage everything from a mobile browser.

## Quick Start

**1. Download the binary for your platform:**

| Binary | Platform |
|--------|----------|
| `ojiiserve-darwin-arm64` | macOS Apple Silicon (M1/M2/M3/M4) |
| `ojiiserve-darwin-x64` | macOS Intel |
| `ojiiserve-linux-arm64` | Linux ARM64 (Android/Termux, Raspberry Pi) |
| `ojiiserve-linux-x64` | Linux x64 (old PCs, servers, VPS) |

**2. Make it executable and run:**

```bash
chmod +x ojiiserve-*
./ojiiserve-linux-arm64  # replace with your platform binary
```

**3. Open the dashboard:**

Visit `http://localhost:9000` in your browser. The setup wizard will walk you through naming your node, downloading dependencies, and registering your admin passkey.

---

## Platform Guides

### macOS (Apple Silicon)

```bash
curl -fsSL -o ojiiserve https://github.com/Ojii-Information-Techonology-Solutions/ojiiserve-binary-releases/releases/latest/download/ojiiserve-darwin-arm64
chmod +x ojiiserve
./ojiiserve
```

Open `http://localhost:9000` in Safari or Chrome.

### macOS (Intel)

```bash
curl -fsSL -o ojiiserve https://github.com/Ojii-Information-Techonology-Solutions/ojiiserve-binary-releases/releases/latest/download/ojiiserve-darwin-x64
chmod +x ojiiserve
./ojiiserve
```

### Android (Termux)

Install [Termux](https://f-droid.org/en/packages/com.termux/) from F-Droid (not the Play Store version).

```bash
pkg update && pkg upgrade
curl -fsSL -o ojiiserve https://github.com/Ojii-Information-Techonology-Solutions/ojiiserve-releases/releases/download/v1.0.1/ojiiserve-linux-arm64
chmod +x ojiiserve
./ojiiserve
```

Open Chrome on the same phone and go to `http://localhost:9000`.

> **Tip:** To keep Ojii Serve running in the background, use `termux-wake-lock` before starting, or run with `nohup ./ojiiserve &`.

### Linux x64 (Old PCs, Servers, VPS)

```bash
curl -fsSL -o ojiiserve https://github.com/Ojii-Information-Techonology-Solutions/ojiiserve-binary-releases/releases/latest/download/ojiiserve-linux-x64
chmod +x ojiiserve
./ojiiserve
```

### Raspberry Pi / Linux ARM64

```bash
curl -fsSL -o ojiiserve https://github.com/Ojii-Information-Techonology-Solutions/ojiiserve-binary-releases/releases/latest/download/ojiiserve-linux-arm64
chmod +x ojiiserve
./ojiiserve
```

---

## What Happens on First Run

1. You name your node
2. Ojii Serve downloads `sqld` (LibSQL server) and `cloudflared` automatically
3. You register an admin passkey (biometric or security key — no passwords)
4. The dashboard is ready

All data is stored in `~/.ojiiserve/`.

## Ports

| Port | Service |
|------|---------|
| 9000 | Web dashboard |
| 9001 | Internal cluster database |
| 8080+ | LibSQL database instances (assigned per database) |

## Accessing from Other Devices

On the same local network, find your machine's IP and visit `http://<ip>:9000`.

For internet access, configure a Cloudflare Tunnel from the dashboard — no port forwarding needed.

## Running as a Service

### Linux (systemd)

For servers, old PCs, and Raspberry Pi — start on boot and auto-restart on crash.

**1. Move the binary to a system location:**

```bash
sudo mv ojiiserve /usr/local/bin/ojiiserve
```

**2. Create the service file:**

```bash
sudo tee /etc/systemd/system/ojiiserve.service > /dev/null << 'EOF'
[Unit]
Description=Ojii Serve
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
ExecStart=/usr/local/bin/ojiiserve
Restart=on-failure
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
EOF
```

Replace `YOUR_USERNAME` with the user that ran the initial setup (the one whose `~/.ojiiserve/` has the data).

**3. Enable and start:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable ojiiserve
sudo systemctl start ojiiserve
```

**Useful commands:**

```bash
sudo systemctl status ojiiserve    # check status
sudo systemctl restart ojiiserve   # restart
sudo journalctl -u ojiiserve -f    # follow logs
```

### macOS (launchd)

```bash
mkdir -p ~/Library/LaunchAgents

cat > ~/Library/LaunchAgents/tech.ojii.serve.plist << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>tech.ojii.serve</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/ojiiserve</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/ojiiserve.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/ojiiserve.err</string>
</dict>
</plist>
EOF

launchctl load ~/Library/LaunchAgents/tech.ojii.serve.plist
```

**Useful commands:**

```bash
launchctl list | grep ojii          # check if running
launchctl unload ~/Library/LaunchAgents/tech.ojii.serve.plist  # stop
launchctl load ~/Library/LaunchAgents/tech.ojii.serve.plist    # start
```

### Android (Termux)

Termux doesn't have systemd. Use `termux-services` for boot-start, or keep it simple:

**Option A — Simple background run:**

```bash
termux-wake-lock
nohup ./ojiiserve > ojiiserve.log 2>&1 &
```

**Option B — termux-services (starts on Termux boot):**

```bash
pkg install termux-services

mkdir -p ~/.termux/boot
cat > ~/.termux/boot/ojiiserve << 'EOF'
#!/data/data/com.termux/files/usr/bin/sh
termux-wake-lock
cd ~
nohup ./ojiiserve > ojiiserve.log 2>&1 &
EOF
chmod +x ~/.termux/boot/ojiiserve
```

> **Note:** Install the [Termux:Boot](https://f-droid.org/en/packages/com.termux.boot/) app from F-Droid and open it once to enable boot scripts.

---

## Data Location

All configuration, databases, and keys are stored in:

```
~/.ojiiserve/
  bin/          # sqld and cloudflared binaries
  data/         # database files
  keys/         # encryption keys
  logs/         # process logs
  cloudflared/  # tunnel config and certs
  local.db      # node config database
```

## Uninstall

Stop the process and remove the data directory:

```bash
rm -rf ~/.ojiiserve
rm ojiiserve
```
