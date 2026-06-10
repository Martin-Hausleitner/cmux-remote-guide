# 🚀 Cmux Remote: The Ultimate macOS & iOS Setup Guide 📱💻

Willkommen zum ultimativen Setup-Guide für **Cmux Remote**! 
Diese Anleitung zeigt dir Schritt-für-Schritt, wie du die iOS-App mit deinem Mac verbindest, den nativen Swift-Daemon als Hintergrunddienst einrichtest und **Tailscale-Routing-Probleme** elegant umschiffst.

---

## 🍏 1. macOS: Den nativen Relay-Daemon installieren

Damit die iOS-App flüssig und zuverlässig mit deinem Terminal kommuniziert, kompilieren wir den nativen Swift-Relay und richten ihn als `launchd`-Service ein.

1. **Terminal öffnen** und das Repository klonen:
   ```bash
   git clone https://github.com/NewTurn2017/cmux-remote.git /tmp/cmux-remote-install
   cd /tmp/cmux-remote-install
   ```
2. **Installations-Skript ausführen**:
   ```bash
   ./scripts/install-launchd.sh
   ```
   *(Das Skript baut den Swift-Code, schiebt ihn nach `~/.cmuxremote/bin/cmux-relay` und legt eine plist in `~/Library/LaunchAgents` an).*

---

## ⚙️ 2. Konfiguration (`relay.json`) erstellen

Der Daemon stürzt ohne eine gültige Konfiguration ab. Wir erstellen nun die Basis-Config.

Öffne (oder erstelle) die Datei `~/.cmuxremote/relay.json` und füge folgendes JSON ein:

```json
{
  "listen": "0.0.0.0:4399",
  "default_fps": 15,
  "idle_fps": 5,
  "allow_login": ["deine.email@servas.ai"],
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
> 💡 **Wichtig:** Tausche `deine.email@servas.ai` gegen deine echte Tailscale-Login-E-Mail aus! Der `apns`-Block muss zwingend vorhanden sein (Dummy-Werte reichen für lokalen Gebrauch).

---

## 🛡️ 3. LAN Bypass & Auto-Start (WLAN-Direktverbindung)

Standardmäßig blockt der Relay alle Verbindungen, die nicht aus dem Tailscale-Tunnel kommen. Damit die Verbindung auch **direkt im lokalen WLAN** rasant schnell klappt, erlauben wir LAN-IPs.

1. **Launchd-Datei bearbeiten:**
   ```bash
   nano ~/Library/LaunchAgents/com.genie.cmuxremote.plist
   ```
2. **Wert ändern:** Suche nach `CMUX_DEV_ALLOW_LOCALHOST` und setze den Wert auf `1`:
   ```xml
   <key>CMUX_DEV_ALLOW_LOCALHOST</key>
   <string>1</string>
   ```
3. **Service neu starten:**
   ```bash
   launchctl kickstart -k gui/501/com.genie.cmuxremote
   ```
   *🎉 Dein Daemon läuft nun auf Port 4399 und startet nach jedem Mac-Neustart automatisch.*

---

## 🌐 4. Tailscale & ACLs (Der Endgegner) 🐉

Wenn du unterwegs bist, willst du dich über **Tailscale** verbinden. Hier lauern zwei fiese Fallen:

### Falle A: Tailscale Versionen-Chaos
Wenn du Tailscale über **Homebrew** UND den **Mac App Store** installiert hast, beißt sich das.
👉 **Lösung:** Beende die Homebrew-Version (`brew services stop tailscale`) und nutze **ausschließlich** die App aus dem Mac App Store!

### Falle B: Tailscale Access Control Lists (ACLs)
Standardmäßig droppt Tailscale den eingehenden TCP-Traffic auf dem Mac, wenn keine passende Regel existiert. Das Paket kommt am Mac an, aber der Tailscale-Daemon wirft es weg (Fehler: `no rules matched`).

👉 **Lösung:** Dein Tailscale-Admin muss im [Tailscale Admin Dashboard](https://login.tailscale.com/admin/acls) folgende Regel zu den `acls` hinzufügen:

```json
{
  "action": "accept",
  "src": ["deine.email@servas.ai"],
  "dst": ["deine.email@servas.ai:*"]
}
```
*(Das erlaubt all deinen Geräten, untereinander auf beliebigen Ports zu kommunizieren).*

---

## 📱 5. iOS App: Cmux Remote einrichten

Die iOS-App nutzt ein geniales "Zero-Config" Security-Modell: Es gibt **keine** Eingabefelder für User, Passwörter oder API-Keys. Die Authentifizierung erfolgt unsichtbar im Hintergrund über Tailscale!

**Schritt-für-Schritt Verbindung:**
1. **Tailscale aktivieren:** Stelle sicher, dass Tailscale auf dem iPhone aktiv ist und sich im selben Netz wie dein Mac befindet.
2. **Öffne die "cmux Remote" App** auf dem iPhone.
3. Scrolle in den Einstellungen zum Bereich **Mac 연결** (Mac Connection).
4. Trage bei **Host** deine IP ein:
   * **Zuhause (WLAN):** Deine lokale Mac-IP (z.B. `192.168.x.x`), falls du den LAN Bypass (Schritt 3) aktiviert hast.
   * **Unterwegs / Standard:** Deine Tailscale-IP (z.B. `100.x.y.z`) oder dein Tailnet-DNS-Name.
5. Trage bei **Port** `4399` ein (dies ist der Standard-Port der `relay.json`).
6. Tippe auf **저장 후 연결 다시 시도** (Save & Reconnect).

**🔑 Wie funktioniert das automatische Pairing?**
Beim ersten Verbindungsaufbau ruft die App den Endpunkt `/v1/devices/me/register` deines Mac-Daemons auf. Der Daemon prüft über Tailscale deine Identität, gleicht sie mit dem Feld `allow_login` in der `relay.json` ab und stellt dem iPhone automatisch ein sicheres Device-Token (Bearer-Token) aus. Dieses wird im Keychain deines iPhones gespeichert.
Wenn du die Verbindung trennen willst, klickst du in der App einfach auf den roten Button **"[ UNPAIR THIS DEVICE ]"**.

✅ **Fertig!** Sobald der Status auf `connected` springt und deine "Workspaces" sichtbar werden, hast du dein Terminal direkt auf dem iPhone! 🚀🔥
