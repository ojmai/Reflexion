# Reflexion - Product Specification

## Übersicht

**Reflexion** ist eine native Desktop-App für wöchentliche Selbstreflexion. Jeden Freitag führt die App den Nutzer durch eine kurze Reflexions-Session, trackt Fortschritte über Zeit und erstellt am Jahresende einen umfassenden Bericht über persönliche Entwicklungen.

---

## Technologie-Stack

| Komponente | Technologie |
|------------|-------------|
| App-Framework | Tauri 2.x (Rust-Backend) |
| Frontend | React 18+ mit TypeScript |
| Styling | Tailwind CSS |
| Datenbank | SQLite (lokal, eine Datei) |
| LLM-Integration | Konfigurierbare API (Claude, OpenAI, etc.) |
| Plattform | macOS (primär), Windows/Linux möglich |

---

## Kernfunktionen

### 1. Wöchentliche Eingabe-Session

**Trigger:** Jeden Freitag (oder manuell aufrufbar)

**Ablauf:**
1. App zeigt zuerst Rückblick auf letzte Woche (was war gut/schlecht)
2. Bei wiederkehrenden Themen: gezielte Nachfrage ("Schlaf war 3 Wochen ein Thema - wie lief es diese Woche damit?")
3. Neue Eingabe: "Was lief diese Woche gut?"
4. Neue Eingabe: "Was lief diese Woche schlecht?"
5. Abschluss mit Zusammenfassung

**Eingabemaske:**
- Minimalistisch, ein Feld nach dem anderen
- Navigation mit Tab oder Enter
- Große Textfelder für Freitext-Eingabe (Spracheingabe erfolgt extern)
- Kein visueller Overhead, Fokus auf Inhalt

### 2. Automatische Kategorisierung

**Zeitpunkt:** Nach Abschluss der Eingabe (Batch, nicht live)

**Funktion:**
- LLM analysiert die Freitext-Eingaben
- Extrahiert Kategorien wie: Schlaf, Sport, Arbeit, Beziehungen, Gesundheit, Produktivität, etc.
- Speichert Kategorien zur Eingabe in der Datenbank
- Erkennt Muster über mehrere Wochen

### 3. Intelligente Follow-ups

**Logik:**
- Wenn ein Thema 2+ Wochen hintereinander als "schlecht" kategorisiert wird, wird es zum Follow-up-Thema
- Bei der nächsten Session fragt die App gezielt nach diesem Thema
- Hartnäckigkeit steigt mit Wiederholungen ("Das ist jetzt die 4. Woche mit Schlafproblemen...")
- Optional: Vorschlag, ein konkretes Ziel zu setzen

### 4. Dashboard

**Inhalte:**
- Übersicht der letzten Wochen (Kalender oder Liste)
- Kategorie-Verteilung (welche Themen wie oft gut/schlecht)
- Aktuelle Streak-Anzeige
- Schnellzugriff auf vergangene Einträge
- Zugang zum Jahresbericht (wenn Daten vorhanden)

**Design:**
- Dunkel, modern, minimal
- Klare Typografie
- Subtile Animationen
- Keine überladenen Charts, eher einfache Visualisierungen

### 5. Streak-Tracking / Gamification

**Mechanik:**
- Zählt aufeinanderfolgende Wochen mit abgeschlossener Reflexion
- Visuelle Anzeige im Dashboard ("12 Wochen in Folge")
- Optionale Milestone-Markierungen (4 Wochen, 12 Wochen, 26 Wochen, 52 Wochen)

### 6. Jahresbericht

**Trigger:** Manuell aufrufbar, prominent ab Dezember

**Inhalt:**
- Zusammenfassung aller 52 (oder weniger) Wochen
- Analyse: Welche Themen waren dominant?
- Positive Veränderungen über das Jahr
- Negative Veränderungen / hartnäckige Probleme
- Highlights und Tiefpunkte
- Visualisierung der Entwicklung pro Kategorie

**Export:**
- In-App Anzeige
- Export als PDF
- Export als Markdown

---

## Datenmodell (SQLite)

### Tabelle: entries

```
id: INTEGER PRIMARY KEY
created_at: DATETIME
week_number: INTEGER
year: INTEGER
good_text: TEXT (Roher Eingabetext - was lief gut)
bad_text: TEXT (Roher Eingabetext - was lief schlecht)
summary: TEXT (LLM-generierte Zusammenfassung)
processed: BOOLEAN (LLM-Verarbeitung abgeschlossen?)
```

### Tabelle: categories

```
id: INTEGER PRIMARY KEY
entry_id: INTEGER (Foreign Key zu entries)
name: TEXT (z.B. "Schlaf", "Sport")
sentiment: TEXT ("positive" oder "negative")
```

### Tabelle: follow_ups

```
id: INTEGER PRIMARY KEY
category_name: TEXT
consecutive_weeks: INTEGER
is_active: BOOLEAN
created_at: DATETIME
resolved_at: DATETIME (nullable)
```

### Tabelle: streaks

```
id: INTEGER PRIMARY KEY
current_streak: INTEGER
longest_streak: INTEGER
last_entry_date: DATE
```

### Tabelle: settings

```
key: TEXT PRIMARY KEY
value: TEXT
```

---

## LLM-Integration

### Konfiguration

- API-Key wird in `.env`-Datei gespeichert
- Unterstützte Variablen:
  - `LLM_PROVIDER` (claude, openai, ollama)
  - `LLM_API_KEY`
  - `LLM_MODEL` (optional, sonst Default pro Provider)
  - `LLM_BASE_URL` (optional, für self-hosted)

### Fallback-Verhalten

Wenn kein API-Key konfiguriert:
- Eingabe funktioniert normal
- Kategorisierung deaktiviert
- Follow-ups deaktiviert
- Jahresbericht zeigt nur Rohdaten ohne Analyse
- Hinweis in der App: "LLM nicht konfiguriert - erweiterte Features deaktiviert"

### LLM-Aufrufe

**Nach Eingabe-Session:**
```
Prompt: Analysiere diese wöchentliche Reflexion.

Gut gelaufen:
[GOOD_TEXT]

Schlecht gelaufen:
[BAD_TEXT]

Aufgaben:
1. Erstelle eine kurze Zusammenfassung (2-3 Sätze)
2. Extrahiere Kategorien mit Sentiment (positive/negative)
3. Identifiziere Themen die Nachverfolgung brauchen

Antworte auf Deutsch im JSON-Format.
```

**Vor nächster Session:**
```
Prompt: Basierend auf diesen vergangenen Einträgen, generiere Follow-up-Fragen.

Letzte Woche:
[LAST_WEEK_SUMMARY]

Wiederkehrende Themen (negativ):
[RECURRING_NEGATIVE_CATEGORIES]

Erstelle 1-3 gezielte Nachfragen auf Deutsch.
```

**Jahresbericht:**
```
Prompt: Erstelle einen Jahresbericht basierend auf diesen wöchentlichen Reflexionen.

[ALLE_ENTRIES_DES_JAHRES]

Analysiere:
- Dominante Themen
- Positive Entwicklungen über das Jahr
- Hartnäckige Probleme
- Wendepunkte
- Gesamtfazit

Schreibe auf Deutsch, strukturiert, persönlich aber nicht kitschig.
```

---

## User Interface

### Ansicht 1: Eingabe-Modus

**Layout:**
- Vollbild, zentrierter Content
- Maximal ein Element im Fokus
- Große, lesbare Schrift
- Dunkler Hintergrund, helle Schrift

**Flow:**
1. Begrüßung + Streak-Info ("Woche 12 in Folge!")
2. Rückblick letzte Woche (wenn vorhanden)
3. Follow-up-Fragen (wenn vorhanden)
4. Eingabefeld "Was lief diese Woche gut?"
5. Eingabefeld "Was lief diese Woche schlecht?"
6. Bestätigung + Zusammenfassung

**Interaktion:**
- Enter oder Tab zum Weiter
- Escape zum Zurück
- Cmd+Enter zum Abschließen

### Ansicht 2: Dashboard

**Layout:**
- Sidebar oder Top-Navigation
- Hauptbereich mit Widgets/Cards

**Widgets:**
- **Streak-Card:** Aktuelle Serie, längste Serie
- **Letzte Einträge:** Liste der letzten 4-8 Wochen, klickbar
- **Kategorien-Übersicht:** Welche Themen wie oft (simple Balken oder Tags)
- **Jahresbericht-Button:** Prominent wenn Jahr fast voll

**Interaktion:**
- Klick auf vergangene Woche öffnet Detail-Ansicht
- Jahresbericht öffnet eigene Ansicht

### Ansicht 3: Jahresbericht

**Layout:**
- Scroll-basiert, wie ein Dokument
- Klare Abschnitte mit Überschriften
- Visualisierungen eingebettet

**Aktionen:**
- "Als PDF exportieren"
- "Als Markdown exportieren"

---

## Design-System

### Farben

- **Hintergrund:** Sehr dunkles Grau (#0a0a0a bis #1a1a1a)
- **Oberflächen:** Leicht helleres Grau (#2a2a2a)
- **Text primär:** Weiß (#ffffff)
- **Text sekundär:** Gedämpftes Grau (#888888)
- **Akzent:** Subtiles Blau oder Grün für positive Elemente
- **Warnung:** Gedämpftes Orange für negative Trends

### Typografie

- **Font:** System-Font (San Francisco auf Mac)
- **Größen:** Großzügig, mindestens 16px Basis
- **Überschriften:** Bold, deutlich abgesetzt

### Spacing

- Großzügige Abstände
- Viel Whitespace
- Karten mit Padding, nicht gedrängt

### Animationen

- Subtile Fade-Ins
- Sanfte Übergänge zwischen Ansichten
- Keine ablenkenden Effekte

---

## Projektstruktur

```
reflexion/
├── src-tauri/           # Rust Backend
│   ├── src/
│   │   ├── main.rs
│   │   ├── database.rs  # SQLite Operationen
│   │   ├── llm.rs       # LLM API Calls
│   │   └── commands.rs  # Tauri Commands
│   ├── Cargo.toml
│   └── tauri.conf.json
├── src/                 # React Frontend
│   ├── components/
│   │   ├── InputSession/
│   │   ├── Dashboard/
│   │   ├── YearReport/
│   │   └── shared/
│   ├── hooks/
│   ├── stores/          # State Management
│   ├── styles/
│   ├── App.tsx
│   └── main.tsx
├── .env                 # LLM API Keys (nicht in Git)
├── .env.example         # Template für .env
├── package.json
└── README.md
```

---

## Entwicklungs-Reihenfolge

### Phase 1: Grundgerüst
1. Tauri-Projekt aufsetzen mit React + TypeScript + Tailwind
2. SQLite-Datenbank initialisieren
3. Basis-UI mit Navigation zwischen Eingabe und Dashboard

### Phase 2: Kernfunktion
4. Eingabe-Flow implementieren (ohne LLM)
5. Daten speichern und laden
6. Dashboard mit echten Daten

### Phase 3: LLM-Integration
7. .env Konfiguration
8. LLM-Calls für Kategorisierung
9. Follow-up-Logik

### Phase 4: Polish
10. Streak-Tracking
11. Jahresbericht
12. PDF/Markdown Export
13. Design-Feinschliff

---

## Zusätzliche Hinweise

### Offline-Fähigkeit
Die App funktioniert grundsätzlich offline. LLM-Features werden nachgeholt sobald Verbindung besteht.

### Datenschutz
Alle Daten bleiben lokal. Nur die LLM-API-Calls gehen nach außen. Nutzer sollte darauf hingewiesen werden.

### Backup
SQLite-Datei kann manuell kopiert werden. Optional: Export aller Daten als JSON.

### Erweiterbarkeit
Das System ist so gebaut, dass später weitere Features ergänzt werden können:
- Explizite Ziele mit Tracking
- Monatliche Zusammenfassungen
- Reminder-Notifications
- Mehrere Jahre vergleichen