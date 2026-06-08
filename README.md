# Cmux Remote on macOS with Tailscale

A complete guide from start to finish on setting up the Cmux Remote iOS app with a macOS host, specifically handling Tailscale and network routing issues.

## 1. Install the Relay Daemon
First, clone the native Swift relay and compile it. The native Swift relay is required for the iOS app to correctly mirror the tmux session.

```bash
git clone https://github.com/GenieApp/cmux-remote.git /tmp/cmux-remote-install
cd /tmp/cmux-remote-install
./scripts/install-launchd.sh
```

## 2. Configure `relay.json`
The config file `~/.cmuxremote/relay.json` needs to be valid JSON. An empty or invalid config will crash the daemon.

Create `~/.cmuxremote/relay.json`:
```json
{
  "listen": "0.0.0.0:4399",
  "default_fps": 15,
  "idle_fps": 5,
  "allow_login": ["your_tailscale_email@domain.com"],
  "apns": {
    "key_path": "",
    "key_id": "DUMMY00001",
    "team_id": "DUMMY00001",
    "topic": "com.genie.cmuxremote",
    "env": "prod"
  },
  "snippets": []
}
```

## 3. Enable LAN Bypass
By default, the relay relies on `tailscaled` to authenticate connections via `tailscale whois`. If your iPhone connects directly over local WiFi, the Tailscale Daemon will drop the packets or return "unauthorized". 
To fix this, edit the launchd plist:

```bash
nano ~/Library/LaunchAgents/com.genie.cmuxremote.plist
```
Change `<key>CMUX_DEV_ALLOW_LOCALHOST</key>` from `<string>0</string>` to `<string>1</string>`.

Restart the daemon:
```bash
launchctl kickstart -k gui/501/com.genie.cmuxremote
```

## 4. Tailscale ACLs (CRITICAL)
If you're connecting via Tailscale, your Tailnet ACLs must permit TCP traffic on port 4399. Otherwise, the local Tailscale network extension on macOS will drop incoming packets with the error `no rules matched`.

In your Tailscale Admin Console -> Access Controls, add a rule allowing your devices to reach port 4399:

```json
{
  "action": "accept",
  "src": ["your_email@domain.com"],
  "dst": ["your_email@domain.com:*"]
}
```

## 5. Multiple Tailscale Instances Conflict
If you have installed Tailscale via Homebrew AND the Mac App Store, you will encounter routing issues, causing DERP timeouts or direct connection failures.
Ensure only the Mac App Store version is running:

```bash
brew services stop tailscale
```
And launch Tailscale from your Applications folder (or Menubar).

## 6. Connect from iOS
In the Cmux Remote app:
- **Host:** `100.x.y.z` (your Mac's Tailscale IP) or `10.x.y.z` (your Mac's Local WiFi IP)
- **Port:** `4399`

Tap **SAVE & RECONNECT**.
