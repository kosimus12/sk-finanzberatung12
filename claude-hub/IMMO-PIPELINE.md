# IMMO-Pipeline – Playbook (Scraper → Analyse → Freigabe → Anschreiben)

Dieses Playbook verbindet deinen **Immobilien-Scraper** mit **Alex** über den
Claude Hub. Es setzt auf deinem bestehenden Setup auf (Drive-Ordner
„Alex-Gedächtnis": *Alex-Stammdaten*, *Alex-Assistenzregeln*,
*gym_property_validation_schema.json*) und verfeinert den IMMO-Workflow um deine
neuen Vorgaben (unterschiedliche Absender/Personas + Telegram-Freigabe).

> **Grundregel (aus deinen Sicherheitsstufen):** Nichts geht ohne deine Freigabe
> raus. Mailversand IMMER über `propose_send` → Telegram **OK/Nein**. Kein
> `send_message confirm=true` ohne ausdrückliche Einzelfreigabe.

## Der automatische Ablauf

```
[Hetzner-Claude / Opus 4.6]                [Mac-Claude "Alex" / Opus 4.8]        [Du]
  1 Scraper läuft (Cron)                                                          
  2 Rohtreffer + Enrichment  ──Hub──▶  3 Kategorisieren & Bewerten               
    (Places/OSM: Wettbewerb,             4 Interessante Treffer je Kategorie      
     Einzugsgebiet, Unterversorgung)        → Entwurf Anschreiben (richtiger      
                                              Absender/Persona)                    
                                          5 Telegram: Angebot + Kennzahlen  ──▶  6 "ja"/"nein"
                                          7 bei "ja": Mail raus (propose_send)     
```

**Delegation (Tokens sparen):** Die schwere Arbeit (Scrapen, Enrichment,
Scoring über viele Objekte) läuft auf dem **Hetzner-Opus-4.6**. Nur die finale
Bewertung, das Anschreiben und die Freigabe-Kommunikation macht **Alex auf dem
Mac (Opus 4.8)**. Übergabe der Ergebnisse läuft über den Hub-Nachrichten-Bus.

## Die 4 Kategorien – Absender & Persona

| # | Kategorie | Zweck | Persona | Absender-Konto |
|---|---|---|---|---|
| **a** | Privat **Miete** | Wohnung, in der du selbst wohnen willst | **Du als Simon** (Ich-Form, privat) | `simon.kuper97@gmail.com` (k97) |
| **b** | Privat **Kauf zum Selbstbewohnen** | Immobilie zum Selbstnutzen kaufen | **Du als Simon** (Ich-Form, privat) | `simon.kuper97@gmail.com` (k97) |
| **c** | Kauf **zum Vermieten** | Kapitalanlage/Vermietung | **Alex, Assistent von Simon** | `alex@sk-finanzberatung.de` |
| **d** | Gewerbe-**Miete Smart Gym** | Fläche fürs 24/7-Smart-Gym | **Alex, Assistent von Simon** | `alex@sk-finanzberatung.de` |

> Wichtigste Änderung ggü. den alten Assistenzregeln: Bei **a/b** meldet sich
> Alex **nicht** als Assistent, sondern schreibt **als du** von deiner privaten
> Adresse. Bei **c/d** bleibt es bei „Alex, Assistent von Simon" über die
> SK-Finanzberatung-Adresse.

## Analyse-Regeln je Kategorie

**a/b – Privat (Miete/Kauf zum Wohnen):** Filter nach deinen privaten Wohn-
Kriterien (Region, Budget, Zimmer, Lage). *(Diese Kriterien sind noch nicht
hinterlegt – siehe „Offene Punkte".)*

**c – Kauf zum Vermieten:** Standard-Kapitalanlage-Kennzahlen: Kaufpreis,
Kaltmiete, **Brutto-Mietrendite**, Preis/m², Lage/Mikrolage, Zustand,
Nebenkostenrisiko. Nur Objekte mit plausibler Rendite vorschlagen.

**d – Smart Gym:** Strikt nach `gym_property_validation_schema.json`:
- **Knockout** (ein Verstoß → verwerfen): Fläche 200–750 m², Deckenhöhe ≥ 2,8 m,
  Bodenlast ≥ 500 kg/m², Etage EG/Hochparterre (OG nur mit Lastenaufzug),
  Miete ≤ 12 €/m² kalt und ≤ 4.500 € gesamt, Gebietstyp GE/GI/MI/MK/SO
  (kein reines Wohngebiet wegen TA-Lärm 24/7), Nutzung zulässig, bezugsfertig.
- **Wirtschaft-Gate:** je Objekt max. tragbare Kaltmiete rechnen (EK 40.000 €,
  Beitrag 34,90/39,90 €, Auslastung 0,70, 1,1–1,3 Mitglieder/m²).
- **Scoring 0–100** + **Unterversorgung** (Einwohner je Studio > 9.160 = HOT).
- Zielregionen: **Kiel + Umland**, **Dortmund + Umland**.
- Fehlende Werte (Deckenhöhe/Bodenlast/Gebietstyp fehlen auf Immoscout oft!)
  → Status **„prüfen"** → an dich/Alex, **nicht** verwerfen.

## Täglicher Betrieb (dein Rhythmus)

- **Uhrzeit:** Scraper **+ Alex-Validierung** laufen **täglich um 16:00**. So hast
  du nachmittags/abends Zeit, bei Bedarf noch zu telefonieren.
- **Erster Lauf = Voll-Scan** über alle passenden Inserate. **Danach nur Delta:**
  jeden Tag nur **neu hinzugekommene** Objekte prüfen (bereits gesehene IDs in
  einer „seen"-Liste auf Hetzner speichern und überspringen). Spart Zeit + Tokens.
- **Bilder + Exposé-PDFs immer mitverwerten:** Alex lädt zu jedem interessanten
  Objekt die **Fotos und das Exposé-PDF**, wertet sie per Bild-/Textanalyse aus
  (Zustand, Grundriss, Deckenhöhe/Boden-Hinweise, versteckte Mängel, Ausstattung)
  und lässt das in die Einschätzung einfließen.
- **Kontext:** Alex zieht deinen Kontext aus dem gemeinsamen Gedächtnis / Cowork
  (Budget, Regionen, Smart-Gym-Kriterien), um treffsicher zu bewerten.

## Telegram-Freigabe (das „einmal kurz validieren")

Pro interessantem Treffer **eine kurze** Telegram-Nachricht — **nicht zu viel**,
genau: **Immobilie · Einschätzung · Mini-Zusammenfassung**:

```
🏠 [c – Vermieten] Musterstr. 1, 24103 Kiel · 245.000 € · 68 m²
Einschätzung: solide Kapitalanlage, Rendite ~3,5 %, Score 74/100.
Kurz: gepflegt (Bj. 98), 2/4, Fotos ok, Exposé ohne Mängel-Hinweis.
📎 3 Fotos + Exposé ausgewertet · ImmoScout <Link>
JA = anschreiben (alex@sk-finanzberatung.de) · NEIN · DETAILS
```

Du antwortest **JA / NEIN** (oder DETAILS für mehr) direkt im Telegram-Chat
(läuft über den Hub → Mac-Alex → `propose_send`). Erst nach **„JA"** geht die
Kontaktnachricht über das richtige Konto/Persona raus.

## So wird es „scharf geschaltet"

1. **Hub deployen + Brücken starten** (siehe `README.md`) – Voraussetzung für
   Telegram-Freigabe und Agenten-Kommunikation.
2. **Scraper auf Hetzner als täglichen Cron um 16:00** einrichten (Beispiel):
   ```cron
   0 16 * * *  /usr/bin/node /opt/immo-scraper/run.js >> /var/log/immo.log 2>&1
   ```
   Der Lauf: (a) neue Inserate holen, (b) gegen `seen.json` filtern (nur Delta),
   (c) Smart-Gym-Objekte gegen `gym_property_validation_schema.json` scoren,
   (d) Ergebnis (inkl. Foto-/PDF-Links) an den Hub melden:
   `POST /message {toCapability:"main", body:"IMMO 16:00: <json/link>"}`.
   Danach triggert Alex automatisch Analyse → Telegram-Digest.
3. **Alex-Memory ergänzen:** Dieses Playbook als Regel in „Alex-Gedächtnis"
   aufnehmen (oder ins gemeinsame Hub-Gedächtnis), damit alle Agenten es kennen.

> **Ehrlicher Stand (aus deinem Drive/Cockpit):** Deine **Prüflogik**
> (`gym_property_validation_schema.json`, v2.0) und die **Workflow-Regeln** sind
> aktuell und vollständig. Einen **laufenden Immo-Scraper** finde ich aber weder
> in Drive noch in deinem Projekt-Cockpit (dort laufen IG-Engine, LinkedIn-Scraper
> via Apify, OpenClaw-Abbau u. a. — **kein** Immo-Scraper). Der Scraper-Code liegt
> vermutlich lokal auf Mac/Hetzner, worauf ich aus der Cloud **keinen Zugriff**
> habe. Heißt: „täglich um 16:00 einrichten" macht der **Hetzner-Claude** über die
> Brücke — ich liefere hier die genaue Vorlage dafür.

## Offene Punkte (brauche ich von dir)

- **Privat-Kriterien (a/b):** Region(en), Budget Miete/Kauf, Zimmer/Größe, Muss/
  Kann. Für Smart Gym (d) und Vermieten (c) ist alles hinterlegt, für deine
  privaten Wohn-Wünsche noch nicht.
- **Scraper-Quelle:** Nur ImmoScout24 oder auch Kleinanzeigen/Immowelt? Und liegt
  irgendwo schon Scraper-Code (Mac-Ordner „Mac Related"?), den der Hetzner-Claude
  weiterverwenden soll — oder neu bauen?

### Namens-Klarstellung (wichtig fürs Aufräumen)
- **„OpenClaude/Ralf" = OpenClaw + n8n** auf dem Oracle-Server (158.101.171.104).
  Laut deinem Cockpit läuft der **Abbau** bereits („OpenClaw-Abbau"). Oracle soll
  als abgesichertes Toolkit bleiben.
- **„Hermes"** ist dein aktueller **Telegram-/Freigabe-Bot** (Hetzner-Cron). Die
  neue Hub-Telegram-Anbindung **ersetzt** ihn — Hermes deshalb erst abschalten,
  **wenn** der neue Kanal läuft, sonst fehlt kurz der JA/NEIN-Weg.
