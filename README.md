# ioBroker.luxtronik2ws

## Alpha Innotec / Luxtronik 2.1 WebSocket Adapter

[![GitHub](https://img.shields.io/badge/GitHub-alessandrocivi/Luxtronic--V.3.x-blue)](https://github.com/alessandrocivi/Luxtronic-V.3.x)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Firmware](https://img.shields.io/badge/Firmware-v3.81%2B-green)](https://github.com/alessandrocivi/Luxtronic-V.3.x)
[![ioBroker](https://img.shields.io/badge/ioBroker-Adapter-orange)](https://www.iobroker.net)

---

## Warum ein neuer Adapter?

Der bisherige ioBroker Adapter [UncleSamSwiss/ioBroker.luxtronik2](https://github.com/UncleSamSwiss/ioBroker.luxtronik2) war lange Zeit die Standardlösung für die Integration von Alpha Innotec Wärmepumpen in ioBroker. Leider wird dieser Adapter nicht mehr aktiv weiterentwickelt und funktioniert mit neueren Firmware-Versionen der Luxtronik 2.1 Steuerung **nicht mehr**.

### Das Problem mit dem alten Adapter

Ab Firmware **V3.81** hat Alpha Innotec das Kommunikationsprotokoll grundlegend geändert:

- Die alte Luxtronik 2.0 Steuerung kommunizierte über ein einfaches **TCP-Socket-Protokoll** auf Port 8888
- Ab Firmware V3.81 wurde dieses Protokoll durch ein modernes **WebSocket-Interface** auf Port 8214 ersetzt
- Das neue WebSocket-Interface verwendet das Subprotokoll `Lux_WS` und eine JSON-basierte Kommunikation
- Der alte Adapter von UncleSamSwiss unterstützt dieses neue Protokoll nicht — die Verbindung schlägt sofort fehl
- Ab Firmware **V3.92.x** ist die Verbindung über das alte Protokoll vollständig blockiert

### Bekannte Probleme mit dem alten Adapter (UncleSamSwiss)

- Verbindung wird sofort getrennt (`connection closed`)
- Keine Daten werden empfangen
- Fehlermeldung: `WebSocket connection failed`
- Adapter zeigt dauerhaft roten Punkt in ioBroker
- Das Repository wird nicht mehr aktiv gepflegt und erhält keine Updates

### Die Lösung: ioBroker.luxtronik2ws

Dieser neue Adapter wurde von Grund auf neu entwickelt und unterstützt das **neue WebSocket-Protokoll** der Luxtronik 2.1 Steuerung ab Firmware V3.81. Er wurde speziell mit Firmware **V3.92.2** getestet und funktioniert zuverlässig.

---

## Unterstützte Geräte

Alle Wärmepumpen mit **Luxtronik 2.1 Steuerung** und Firmware **V3.81 oder neuer**:

- Alpha Innotec (alle Modelle mit Luxtronik 2.1)
- Novelan (baugleich mit Alpha Innotec)
- Siemens Wärmepumpen mit Luxtronik 2.1
- Alle OEM-Varianten der Luxtronik 2.1 Steuerung

**Getestete Firmware:** V3.81, V3.90, V3.91, V3.92.2

> ⚠️ **Für ältere Geräte mit Firmware < V3.81** bitte den ursprünglichen Adapter von [UncleSamSwiss](https://github.com/UncleSamSwiss/ioBroker.luxtronik2) verwenden.

---

## ⚠️ Voraussetzungen — Netzwerk & Webserver aktivieren

> **Wichtig:** Bevor der Adapter verwendet werden kann, muss die Wärmepumpe zwingend am lokalen Netzwerk (LAN) angeschlossen sein und der **Webserver** sowie die **Fernsteuerung** in der Luxtronik-Steuerung aktiviert sein.

### Schritt 1 — LAN-Kabel anschliessen

Die Luxtronik 2.1 Steuerung unterstützt **ausschliesslich Kabelverbindung (LAN)**. WLAN ist nicht unterstützt.

1. LAN-Kabel (RJ-45) an die Netzwerkbuchse der Luxtronik-Steuerung anschliessen
2. Anderes Ende in den Router/Switch einstecken
3. Maximale Kabellänge: 99 Meter

> Quelle: [Alpha Innotec Anleitung Elektriker](https://www.alpha-innotec.ch/fileadmin/content/product_management/alpha_web/Anleitung_Elektriker_de.pdf)

---

### Schritt 2 — Webserver aktivieren

Am Bedienfeld der Wärmepumpe:

```
SERVICE → Systemsteuerung → Webserver → Aktivieren
```

Detaillierte Schritte:
1. **Dreh-Druck-Knopf** drehen bis `SERVICE` markiert ist → drücken
2. `Systemsteuerung` anwählen → drücken
3. `Webserver` anwählen → drücken
4. Webserver auf **„Ein"** stellen → mit Haken bestätigen

> Nach der Aktivierung ist die Wärmepumpe unter ihrer lokalen IP-Adresse im Browser erreichbar, z.B.: `http://192.168.1.67`

> Quelle: [Alpha Innotec Betriebsanleitung Luxtronik 2.0/2.1](https://www.alpha-innotec.ch/fileadmin/content/downloads/Lux_Fachhandwerker_de.pdf) — Seite 62

---

### Schritt 3 — Fernsteuerung aktivieren

```
SERVICE → Einstellung → System Einstellung → Fernsteuerung → Ja
```

Detaillierte Schritte:
1. **Dreh-Druck-Knopf** drehen bis `SERVICE` markiert ist → drücken
2. `Einstellung` anwählen → drücken
3. `System Einstellung` anwählen → drücken
4. Bis `Fernsteuerung` scrollen → `Ja` wählen → mit Haken bestätigen

> ⚠️ Laut offizieller Betriebsanleitung: *„Alle Einstellungen, die die Funktion Fernwartung betreffen, dürfen nur durch autorisiertes Servicepersonal vorgenommen werden."*

> Quelle: [Alpha Innotec Betriebsanleitung Luxtronik](https://www.manualslib.de/manual/593845/Alpha-Innotec-Luxtronik.html) — Seite 32ff

---

### Schritt 4 — IP-Adresse der Wärmepumpe herausfinden

**Option A — DHCP (empfohlen):**
```
SERVICE → Systemsteuerung → DHCP-Client → Aktivieren
```
Die Wärmepumpe bezieht automatisch eine IP-Adresse vom Router.
Die zugewiesene IP-Adresse ist im Router-Interface unter den verbundenen Geräten sichtbar.

**Option B — Feste IP:**
```
SERVICE → Systemsteuerung → IP-Adresse → manuell eingeben
```

> 💡 **Tipp:** Im Router eine **feste DHCP-Reservierung** für die Wärmepumpe einrichten, damit die IP-Adresse immer gleich bleibt.

---

### Schritt 5 — Verbindung testen

Nach der Aktivierung die Verbindung mit PowerShell testen:

```powershell
# Ping Test
ping 192.168.1.67

# Port 8214 Test (WebSocket)
Test-NetConnection -ComputerName 192.168.1.67 -Port 8214
```

Erwartetes Ergebnis: `TcpTestSucceeded : True`

---

## Test-Tool — WebSocket Tester (webtest.html)

Im Ordner `test/` ist eine **webtest.html** enthalten mit der die WebSocket-Verbindung zur Wärmepumpe direkt im Browser getestet werden kann — ohne ioBroker oder Node.js.

### Was kann man mit dem Test-Tool machen?

- ✅ **Verbindung testen** — prüfen ob der WebSocket Port 8214 erreichbar ist
- ✅ **Login testen** — Passwort-Authentifizierung prüfen
- ✅ **Alle Datenbereiche anzeigen** — Navigation der Wärmepumpe wird automatisch geladen
- ✅ **Einzelne Bereiche abfragen** — per Klick auf Schaltfläche Daten eines Bereichs anzeigen
- ✅ **Live-Werte anzeigen** — Temperaturen, Betriebsstunden, Status in Echtzeit sehen
- ✅ **IDs prüfen** — die dynamischen Hexadezimal-IDs der Luxtronik Navigationsstruktur einsehen

### So verwenden:

1. Datei `test/webtest.html` herunterladen
2. **Doppelklick** auf die Datei — öffnet sich direkt im Browser (Edge, Chrome)
   > ⚠️ Die Datei muss als **lokale Datei** (`file://`) geöffnet werden, nicht über einen Webserver — sonst blockiert der Browser die WebSocket-Verbindung (Mixed Content)
3. IP-Adresse, Port und Passwort eingeben
4. **„Verbinden"** klicken

### Benutzeroberfläche:

```
┌─────────────────────────────────────────────────────┐
│  🔥 Luxtronik WebSocket Tester                      │
│  ● Verbunden mit 192.168.1.67:8214                  │
├─────────────────────────────────────────────────────┤
│  IP: [192.168.1.67] Port: [8214] PW: [999999]       │
│  [🔌 Verbinden] [✖ Trennen] [🔄 REFRESH] [🗑 Log]  │
├─────────────────────────────────────────────────────┤
│  📂 BEREICHE (automatisch geladen)                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────┐    │
│  │🌡️        │ │📥        │ │📤               │    │
│  │Tempera-  │ │Eingänge  │ │Ausgänge         │    │
│  │turen     │ │0x11e82d8 │ │0x130b868        │    │
│  │0xc724f8  │ └──────────┘ └──────────────────┘    │
│  └──────────┘                                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────┐    │
│  │🕐        │ │⚡        │ │🏠               │    │
│  │Betriebs- │ │Leistungs-│ │Smart Home       │    │
│  │stunden   │ │aufnahme  │ │Interface        │    │
│  └──────────┘ └──────────┘ └──────────────────┘    │
├─────────────────────────────────────────────────────┤
│  📋 LOG                                              │
│  [20:57:24] ✅ Verbunden mit Lux_WS Protokoll!      │
│  [20:57:24] 📥 Navigation geladen — 16 Bereiche     │
│  [20:57:46] 📤 GET Temperaturen (0xc724f8)          │
│  [20:57:46] 📥 Empfangen: {"items":[...]}           │
├─────────────────────────────────────────────────────┤
│  📊 Temperaturen                                     │
│  ┌─────────────────┐ ┌─────────────────┐           │
│  │ Aussentemperatur│ │ Vorlauftemp.    │           │
│  │   8.5 °C        │ │   35.2 °C       │           │
│  └─────────────────┘ └─────────────────┘           │
└─────────────────────────────────────────────────────┘
```

### Häufige Fehlermeldungen beim Test:

| Fehlermeldung | Ursache | Lösung |
|---|---|---|
| `Fehler: [object Event]` | Falsches Protokoll | Datei lokal öffnen (file://) |
| `Insufficient resources` | Browser blockiert ws:// | Datei lokal öffnen statt über https:// |
| `Verbindung getrennt` | Webserver nicht aktiv | Webserver in Luxtronik aktivieren |
| `TcpTestSucceeded: False` | Port geschlossen | Netzwerk prüfen, Kabel kontrollieren |

---

## Features

- ✅ Vollständige WebSocket-Unterstützung mit `Lux_WS` Subprotokoll
- ✅ Automatische Erkennung aller verfügbaren Datenpunkte via Navigation-API
- ✅ Dynamische Erstellung aller ioBroker States — keine manuelle Konfiguration nötig
- ✅ Automatisches Polling aller Werte (konfigurierbar, Standard 30 Sekunden)
- ✅ Automatischer Reconnect bei Verbindungsabbruch
- ✅ Korrekte Einheiten (°C, kWh, kW, h, %)
- ✅ Konfigurierbar via ioBroker Admin UI
- ✅ Verbindungsstatus-Anzeige in ioBroker
- ✅ Kompatibel mit ioBroker MQTT Adapter für Loxone Integration

---

## Installation

### Option 1 — via ioBroker Admin (empfohlen)

1. ioBroker Admin öffnen → **Adapter**
2. Klick auf das **GitHub-Symbol** (Eigene URL)
3. URL eingeben:
   ```
   https://github.com/alessandrocivi/Luxtronic-V.3.x
   ```
4. **Installieren** klicken
5. Instanz anlegen und konfigurieren

### Option 2 — Manuell via Docker

```bash
# In den ioBroker Container einloggen
docker exec -it iobroker bash

# Adapter Verzeichnis erstellen
cd /opt/iobroker/node_modules
mkdir iobroker.luxtronik2ws
cd iobroker.luxtronik2ws

# Alle Dateien aus dem Repository hierher kopieren
# Dann Dependencies installieren:
npm install

# Adapter bei ioBroker registrieren
cd /opt/iobroker
iobroker add luxtronik2ws

# ioBroker neu starten
iobroker restart
```

---

## Konfiguration

| Parameter | Standard | Beschreibung |
|---|---|---|
| **IP Adresse** | 192.168.1.67 | IP-Adresse der Wärmepumpe im lokalen Netzwerk |
| **Port** | 8214 | WebSocket Port (Standard bei allen Luxtronik 2.1) |
| **Passwort** | 999999 | Luxtronik Benutzer-Passwort (Standard: 999999) |
| **Abfrageintervall** | 30s | Wie oft alle Werte abgefragt werden (Sekunden) |
| **Reconnect Intervall** | 60s | Wartezeit bei Verbindungsabbruch (Sekunden) |

---

## Datenpunkte

Der Adapter erstellt **automatisch alle verfügbaren Datenpunkte** basierend auf der Navigation der Wärmepumpe.

| Pfad | Beschreibung | Einheit |
|---|---|---|
| `info.connection` | Verbindungsstatus | boolean |
| `temperaturen.aussentemperatur` | Aussentemperatur | °C |
| `temperaturen.vorlauftemperatur` | Vorlauftemperatur Heizkreis | °C |
| `temperaturen.ruecklauftemperatur` | Rücklauftemperatur Heizkreis | °C |
| `temperaturen.warmwasser_ist` | Warmwasser Isttemperatur | °C |
| `temperaturen.warmwasser_soll` | Warmwasser Solltemperatur | °C |
| `temperaturen.heissgas` | Heissgastemperatur | °C |
| `betriebsstunden.verdichter` | Betriebsstunden Verdichter | h |
| `betriebsstunden.heizung` | Betriebsstunden Heizung | h |
| `anlagenstatus.betriebsstatus` | Aktueller Betriebsstatus | - |
| `energiemonitor.waermemenge` | Erzeugte Wärmemenge gesamt | kWh |
| `energiemonitor.leistungsaufnahme` | Aktuelle Leistungsaufnahme | kW |

---

## Integration mit Loxone

Zusammen mit dem ioBroker MQTT Adapter können alle Wärmepumpen-Daten an einen **Loxone Miniserver** übertragen werden.

1. ioBroker **MQTT Adapter** installieren (Server/Broker Modus, Port 1883)
2. MQTT Adapter: alle `luxtronik2ws.*` States publizieren
3. In **Loxone Config**: MQTT Client hinzufügen → Broker IP = IP des ioBroker Servers, Port 1883
4. Virtuelle Eingänge anlegen, z.B. Topic:
   ```
   luxtronik2ws/0/temperaturen/aussentemperatur
   ```

---

## WebSocket Protokoll (Technische Details)

**Kommunikationsablauf:**
1. Verbinden mit `ws://IP:8214` und Subprotokoll `Lux_WS`
2. Senden: `LOGIN;999999`
3. Empfangen: Navigation JSON mit allen Bereichen und dynamischen IDs
4. Senden: `GET;0xXXXXXX` für jeden Bereich
5. Empfangen: JSON mit allen Werten

> ⚠️ Die IDs in der Navigation sind **dynamisch** — sie können sich nach Firmware-Updates ändern. Der Adapter liest sie bei jedem Start neu aus.

---

## Sicherheitshinweise

> ⚠️ Die Wärmepumpe sollte **niemals direkt aus dem Internet erreichbar** sein. Nur im lokalen Netzwerk verwenden — kein Port-Forwarding einrichten!

> 🔒 In Firmware V3.92 wurde CVE-2024-22894 behoben (hardcodiertes Root-SSH-Passwort). Firmware aktuell halten!

---

## Dateistruktur

```
Luxtronic-V.3.x/
├── main.js                  # Hauptprogramm (ioBroker Adapter)
├── package.json             # Node.js Dependencies
├── io-package.json          # ioBroker Metadaten
├── LICENSE                  # MIT Lizenz
├── README.md                # Diese Dokumentation
├── admin/
│   └── jsonConfig.json      # Konfigurationsseite in ioBroker Admin
├── test/
│   └── webtest.html         # WebSocket Test-Tool für den Browser
└── .github/
    └── workflows/
        └── ci.yml           # GitHub Actions CI
```

---

## Changelog

### 0.1.0 (2026-03)
- Erstveröffentlichung
- WebSocket Verbindung mit `Lux_WS` Protokoll
- Automatische Navigation und Datenpunkt-Erstellung
- Automatisches Polling (konfigurierbar)
- Automatischer Reconnect bei Verbindungsabbruch
- Getestet mit Alpha Innotec Firmware V3.92.2

---

## Mitwirken

Pull Requests und Issues sind herzlich willkommen!

1. Fork erstellen
2. Feature Branch: `git checkout -b feature/mein-feature`
3. Commit: `git commit -m 'Feature hinzugefügt'`
4. Push: `git push origin feature/mein-feature`
5. Pull Request erstellen

---

## Lizenz

MIT License — siehe [LICENSE](LICENSE)

---

## Danksagung

- [UncleSamSwiss](https://github.com/UncleSamSwiss/ioBroker.luxtronik2) für den ursprünglichen Adapter als Inspiration
- Alpha Innotec Community für das Reverse Engineering des `Lux_WS` Protokolls
- ioBroker Community für die hervorragende Plattform
