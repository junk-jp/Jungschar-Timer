# 🏊 Multi-Timer mit Master/Slave-Sync

Timer-System mit steuernder **Master-Seite** und ausgelagerter, per Link verteilbarer **Slave-Anzeige**. Alle Geräte synchronisieren sich in Echtzeit über **Firebase Realtime Database** (kostenloser Spark-Tarif reicht völlig aus). Am Ende jedes Timers sowie am Ende des gesamten Ablaufs ertönt auf allen Geräten ein Signalton.

## Features

- **Master (`index.html`)**
  - Beliebig viele Timer anlegen (Name + Minuten + Sekunden), dauerhaft im Browser gespeichert
  - Ablauf zusammenstellen: Timer per Klick in eine Reihenfolge bringen, umsortieren, entfernen
  - **Loop:** Anzahl der Wiederholungen frei wählbar – der komplette Ablauf läuft so oft durch
  - Start / Pause / Weiter / Überspringen / Stop
  - Live-Anzeige: aktueller Timer, Restzeit, Runde x von y, nächster Timer
  - Room-Code + **Kurzelink der Form `…/r/XXXXXX`** + QR-Code zum Verteilen
- **Slave (`slave.html#room=XXXXXX`, erreicht über `/r/XXXXXX`)**
  - Zeigt den Timer **sofort und ohne Zwischenfenster** an (v2.1)
  - Große Restzeit, Status, Fortschrittsbalken, Runde, nächster Timer
  - Warnt, wenn der Master länger als 6 Sekunden keine Updates sendet
  - Spielt am Ende jedes Timers einen Ton ab: Wird der Ton vom Browser nicht automatisch freigegeben, erscheint nur ein dezenter Hinweisbutton – **ein Tipp irgendwo auf der Seite** schaltet den Ton frei (Browser-Sicherheitsregel)
  - **Ghost-Modus (v2.2):** Fällt die Verbindung zum Master aus (z. B. Handy im Hintergrund, Netzwechsel), läuft die Anzeige lokal weiter und schaltet am Timer-Ende selbstständig zum nächsten Timer weiter – **mit Ton**. Sobald der Master wieder sendet, gleicht sich die Anzeige automatisch zurück
  - **Selbstständige Wiederverbindung (v2.3):** Kehrt ein Slave-Gerät aus dem Hintergrund zurück, baut es die Firebase-Verbindung aktiv neu auf, falls nötig auch per harter Neuverbindung

## Firebase einrichten (einmalig, kostenlos)

1. Kostenloses Google-Projekt auf [console.firebase.google.com](https://console.firebase.google.com) anlegen.
2. Im Projekt **„Realtime Database"** erstellen (Standort beliebig, **Testmodus** wählen).
3. **Projekt-Einstellungen → Allgemein → Ihre Apps → Web-App** hinzufügen und den Abschnitt `firebaseConfig = { … }` kopieren.
4. `index.html` im Browser öffnen → es erscheint ein Einrichtungsfenster → den `firebaseConfig`-Abschnitt einfügen → **Verbinden**.

Die Konfiguration wird lokal im Browser gespeichert. Der Slave braucht nichts mehr einzugeben: Eine Standard-Konfiguration ist **eingebaut** (Projekt `jungschar-timer`), und über den ⚙️-Knopf lässt sich jederzeit eine andere konfigurieren – auch nachträglich auf jedem Slave-Gerät.

### Database-Regeln

Im Testmodus (30 Tage) sind Lesen/Schreiben offen. Für den Dauerbetrieb in der Firebase-Konsole unter *Realtime Database → Regeln* z. B. eintragen:

```json
{
  "rules": {
    "rooms": {
      "$room": {
        ".read": true,
        ".write": true
      }
    }
  }
}
```

Hinweis: Wer den 6-stelligen Room-Code kennt, kann den Timer lesen und steuern. Für nicht-öffentliche Zwecke ist das in der Regel unkritisch; den Code einfach nur mit den eigenen Geräten teilen.

## Slave-Seite öffnen

- Link aus der Master-Seite kopieren (oder QR-Code scannen) und auf den anderen Geräten öffnen – die Anzeige startet sofort.
- Falls der Browser den Ton blockiert, erscheint ein spezieller Hinweisbutton: **einmal irgendwo auf die Seite tippen**, danach spielt das Gerät bei jedem Timer-Ende automatisch mit an.

## Hosting (z. B. GitHub Pages)

Beide Dateien sind komplett eigenständig (nur Firebase-SDK kommt vom CDN).

1. Repository auf GitHub anlegen und `index.html` + `slave.html` + `404.html` hochladen. Die `404.html` ist der Router für die Kurzelinks: `…/r/XXXXXX` landet automatisch auf der Slave-Anzeige mit dem richtigen Room.
2. *Settings → Pages → Source: main branch, root* aktivieren.
3. Aufrufen unter `https://<nutzer>.github.io/<repo>/` (Master) – Slave-Link verteilt sich daraus automatisch.

Alternativ funktioniert jede statische Hosting-Umgebung (Netlify, Vercel, Raspberry Pi mit nginx, …).

## Hinweise

- Timer und Ablauf werden im `localStorage` des Master-Browsers gespeichert – die Master-Seite sollte also dauerhaft im selben Browser offen bleiben, während der Ablauf läuft.
- **Wake Lock (v2.3):** Während ein Ablauf läuft, hält die Master-Seite den Bildschirm wach (Wake Lock API, iOS Safari ab 16.4) – so können Background-Timer nicht mehr einschlafen. Bei Pause/Stop/Ende wird die Sperre automatisch aufgehoben.
- Der Ton wird per Web Audio API erzeugt, es sind keine Audio-Dateien nötig.
- **iOS/iPadOS Safari:** Safari entfernt URL-Parameter (`?room=…`) beim Öffnen von Links aus manchen Apps (Schutz vor Link-Tracking). Deshalb funktionieren die Slave-Links über den **Pfad** (`…/r/XXXXXX`) und merken sich den Room zusätzlich im Browser – falls ein Puffer verloren geht, kann der Room-Code jederzeit über den ⚙️-Knopf neu eingegeben werden.
