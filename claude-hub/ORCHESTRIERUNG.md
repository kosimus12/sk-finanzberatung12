# Orchestrierung & Rollen deiner Claudes

Dieses Dokument beschreibt, wie deine Claudes zusammenarbeiten. Der Text eignet
sich gut als **Startinhalt für das gemeinsame Gedächtnis** (im Dashboard unter
„🧠 Gemeinsames Gedächtnis" einfügen) — dann kennen ihn alle Agenten.

## Die Beteiligten

| Agent | Läuft auf | Modell | Rolle | Fähigkeiten (`CAPABILITIES`) |
|---|---|---|---|---|
| **Haupt-Claude** | Mac | Opus 4.8 (Max-Account) | **Haupt-Ansprechpartner / Orchestrator** | `main,gmail,calendar,drive` |
| **Server-Claude** | Hetzner | Opus 4.6 (Bedrock-API) | Fleiß-Agent für lange/schwere Aufgaben | `worker,github,web` |
| **(optional) Mac-Zweitsitzung** | Mac | Opus 4.8 | Spezialaufgaben | frei wählbar |

> Der **Haupt-Claude** (Fähigkeit `main`) ist deine zentrale Anlaufstelle — auch
> für Telegram. Er verteilt Arbeit an den Server-Claude, wo das sinnvoll ist.

## Token sparen: richtig delegieren

Dein Max-Account (Opus 4.8) soll **wenig Tokens verbrauchen**. Faustregel:

- **Lange, mechanische oder rechenintensive Aufgaben** (Recherche über viele
  Seiten, Massen-Refactorings, Log-Analysen, Stapel-Verarbeitung) →
  **an den Hetzner-Server-Claude (Opus 4.6 / Bedrock) delegieren**. Der läuft
  über die API, nicht über dein Max-Kontingent.
- **Kurze Entscheidungen, Feinschliff, Kundenkommunikation, Orchestrierung** →
  Haupt-Claude (Opus 4.8).

**So delegierst du:**
1. Im Dashboard beim Agenten „Server-Claude" einen Befehl senden, **oder**
2. den Haupt-Claude bitten: „Delegiere das an den Hetzner-Agenten" — er nutzt
   dazu die Brücken-Route `POST /message {toCapability:"worker", body:"…"}`.
3. Über **Coworker** delegierst du an MCP-Dienste (Gmail, Kalender, Drive,
   GitHub …) — der Hub routet die Aktion automatisch an einen Agenten, der die
   Fähigkeit hat.

## Sinnvolle Integration von Server- und Mac-Claude

- **Hetzner (Opus 4.6):** Dauerläufer. Ideal für Aufgaben, die im Hintergrund
  über Stunden laufen dürfen (SEO-Analysen für sk-finanzberatung.de,
  Wettbewerbs-Recherche, Datenaufbereitung, GitHub-Automatisierung). Läuft in
  `tmux`, immer erreichbar, blockiert dein Max-Kontingent nicht.
- **Mac-Claude (Opus 4.8):** Qualitäts- und Orchestrator-Instanz. Beantwortet
  Telegram-Anfragen headless und koordiniert die anderen.
- **Gemeinsames Gedächtnis:** Alle teilen denselben Kontext (Kundenprojekte,
  Tonalität, Fakten zu SK Finanzberatung). Einmal im Dashboard pflegen → wird
  automatisch an alle verteilt.

## Telegram als Hauptkanal

Du schreibst deinem Telegram-Bot → der Hub leitet die Nachricht an den
`main`-Agenten (Mac) → dieser beantwortet sie **headless** (`claude -p …`) →
die Antwort kommt zurück in den Telegram-Chat. Nur deine hinterlegte Chat-ID
darf den Bot steuern (`TELEGRAM_ALLOWED_CHAT`). Einrichtung: siehe `README.md`.

## Alte Tools „Hermes" und „OpenClaude/Ralf" entfernen

Diese laufen auf deinen Maschinen (Hermes nur auf Hetzner, „Ralf/OpenClaude"
auf Mac und/oder Hetzner) und werden nicht mehr gebraucht. **Wichtig:** Aus der
Cloud-Umgebung heraus habe ich keinen Zugriff auf Mac/Hetzner — das Entfernen
läuft auf den Maschinen selbst. Zwei sichere Wege:

**A) Skript (empfohlen, mit Backup):**
```bash
# Erst nur ANSCHAUEN, was gefunden wird (löscht nichts):
bash claude-hub/bridge/cleanup-old-agents.sh
# Dann gezielt entfernen (fragt nach, sichert Konfigs):
bash claude-hub/bridge/cleanup-old-agents.sh --remove
```
Auf Hetzner per SSH ausführen, auf dem Mac lokal. Prüfe vorher die angezeigte
Liste, damit nichts Falsches entfernt wird.

**B) Über den Haupt-Claude orchestrieren:** Sobald die Brücken laufen, kannst du
im Dashboard dem Server-Claude den Befehl geben: „Finde und entferne Hermes und
OpenClaude/Ralf mit `cleanup-old-agents.sh`, zeig mir vorher die Fundliste."
Der Claude auf der Maschine führt das dann unter deiner Aufsicht aus.

> Ich lösche bewusst **nichts blind**: „Hermes" und „Ralf" sind mir inhaltlich
> unbekannt, und ein falscher Löschbefehl wäre nicht umkehrbar. Deshalb erst
> analysieren, dann bestätigen, dann entfernen — mit Backup.
