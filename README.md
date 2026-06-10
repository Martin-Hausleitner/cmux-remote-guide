# 🚀 Cmux Remote: The Ultimate Setup Guide 📱💻

Willkommen zum ultimativen Setup-Guide für **Cmux Remote**! 
Diese Anleitung zeigt dir Schritt-für-Schritt, wie du die offizielle iOS-App (cmux BETA) mit deinem Mac-Terminal verbindest.

---

## 🍏 1. macOS: "Mobile Connect" aktivieren (Die offizielle Methode)

Seit dem cmux-Update vom 6. Juni 2026 (`v0.64.14`) ist der Remote-Daemon **nativ** in die cmux Mac-App integriert. Du brauchst **keine** externen Repositories (`cmux-remote`) oder manuellen `launchd`-Skripte mehr!

1. Öffne die **cmux** App auf deinem Mac.
2. Öffne die **Command Palette** (Befehlspalette).
3. Suche nach **Mobile Connect** und wähle die Option aus, um das Mobile Connect Fenster zu öffnen.
4. Aktiviere in diesem Fenster die Verbindung. Hier siehst du auch den **Pairing Port** (standardmäßig `4399`), den du frei konfigurieren kannst.
5. (Optional) Aktiviere "Forward terminal notifications", um Benachrichtigungen auf dein iPhone zu pushen.

---

## 🌐 2. Tailscale & Netzwerk-Voraussetzungen

Die iOS-App kommuniziert direkt und sicher über **Tailscale**.

1. **Tailscale installieren:** Stelle sicher, dass Tailscale auf deinem Mac **und** iPhone installiert und aktiv ist. (Tipp: Die Mac App Store Version von Tailscale ist oft stabiler als die Homebrew-Version).
2. **Gleiches Tailnet:** Beide Geräte müssen im selben Tailscale-Netzwerk eingeloggt sein.
3. Notiere dir die **Tailscale-IP** deines Macs (z.B. `100.x.y.z`) oder den Tailnet-DNS-Namen.

---

## 📱 3. iOS App: Cmux Remote verbinden

Lade dir die **cmux BETA** iOS-App via TestFlight (oder den App Store) herunter.

**Schritt-für-Schritt Verbindung:**
1. **Tailscale aktivieren:** Überprüfe in der iOS-Statusleiste, dass das VPN-Icon von Tailscale aktiv ist.
2. **Öffne die "cmux" App** auf dem iPhone.
3. Gehe in die Einstellungen (Settings) zum Bereich **Mac Connection** (Mac 연결).
4. Trage bei **Host** deine Tailscale-IP (z.B. `100.x.y.z`) oder den Tailscale DNS ein.
5. Trage bei **Port** deinen konfigurierten Pairing Port ein (Standard: `4399`).
6. Tippe auf **[ SAVE & RECONNECT ]** (저장 후 연결 다시 시도).

**🔑 Sicherheit & Pairing:**
Die Authentifizierung läuft unsichtbar über Tailscale (Zero-Config). Die App kontaktiert den Mac via Tailscale (`/v1/devices/me/register`), und da Tailscale die Identität bereits verifiziert, stellt der Mac automatisch ein sicheres Device-Token aus. Dieses wird im Keychain deines iPhones gespeichert. Es sind **keine SSH-Keys oder Passwörter** nötig!

Um ein Gerät wieder zu entfernen, klicke in den Einstellungen der App auf den roten Button **"[ UNPAIR THIS DEVICE ]"**.

✅ **Fertig!** Sobald der Status auf `connected` springt und deine Workspaces sichtbar werden, hast du dein Terminal direkt auf dem iPhone! 🚀🔥
