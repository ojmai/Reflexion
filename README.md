# Reflexion

Eine native Desktop-App für wöchentliche Selbstreflexion, gebaut mit Tauri 2.x, React 18+, TypeScript und SQLite.

## Features

- 🗓️ **Wöchentliche Reflexionen**: Strukturierter Eingabe-Flow für positive und negative Erlebnisse
- 🤖 **KI-Unterstützung**: Automatische Zusammenfassungen und Kategorisierung mit Claude AI
- 🔥 **Streak-Tracking**: Motivation durch Tracking aufeinanderfolgender Reflexions-Wochen
- 📊 **Follow-ups**: Automatische Erkennung wiederkehrender Themen
- 📈 **Jahresberichte**: KI-generierte Jahresrückblicke mit Highlights und Entwicklungen
- 💾 **Lokale Datenhaltung**: SQLite-Datenbank, keine Cloud, volle Kontrolle
- 🎨 **Dark Mode**: Modernes, augenfreundliches Design

## Voraussetzungen

### System-Anforderungen

- **Node.js** (v18 oder höher)
- **npm** oder **yarn**
- **Rust** (für Tauri)
  - Installation: https://www.rust-lang.org/learn/get-started

### Tauri-Voraussetzungen

Je nach Betriebssystem sind zusätzliche Abhängigkeiten erforderlich:

- **macOS**: Xcode Command Line Tools
- **Linux**: webkit2gtk, libssl, etc. (siehe [Tauri Prerequisites](https://tauri.app/start/prerequisites/))
- **Windows**: Microsoft Visual C++ Build Tools

## Installation

1. **Repository klonen**
   ```bash
   git clone <repository-url>
   cd Reflexion
   ```

2. **Dependencies installieren**
   ```bash
   npm install
   ```

3. **LLM-Konfiguration einrichten** (optional, aber empfohlen)

   Kopiere `.env.example` zu `.env`:
   ```bash
   cp .env.example .env
   ```

   Trage deinen Claude API-Schlüssel ein:
   ```env
   LLM_PROVIDER=claude
   LLM_API_KEY=sk-ant-xxxxx
   ```

   **API-Schlüssel erhalten:**
   - Gehe zu https://console.anthropic.com/
   - Erstelle einen Account und navigiere zu API Keys
   - Erstelle einen neuen API-Schlüssel

## Entwicklung

```bash
npm run tauri dev
```

Dies startet:
- Den Vite Dev-Server (Frontend)
- Die Tauri-App im Development-Modus

Hot-Reload ist für das Frontend aktiviert.

## Build

```bash
npm run tauri build
```

Erstellt eine produktionsreife App für dein Betriebssystem:
- **macOS**: `.dmg` und `.app` in `src-tauri/target/release/bundle/`
- **Windows**: `.exe` und `.msi` Installer
- **Linux**: `.deb`, `.AppImage`, etc.

## Nutzung

### Erste Reflexion erstellen

1. Starte die App
2. Klicke auf "Reflexion starten" im Dashboard
3. Folge dem geführten Eingabe-Flow:
   - Begrüßung mit aktueller Streak
   - Rückblick auf letzte Woche (falls vorhanden)
   - Follow-up Fragen zu wiederkehrenden Themen
   - "Was lief gut?" - beschreibe positive Erlebnisse
   - "Was lief schlecht?" - reflektiere Herausforderungen
   - Zusammenfassung und Bestätigung

### LLM-Features nutzen

Wenn ein API-Schlüssel konfiguriert ist:
- **Automatische Zusammenfassungen**: Nach dem Speichern wird ein kompakter Summary erstellt
- **Kategorisierung**: Positive und negative Themen werden extrahiert
- **Follow-up Erkennung**: Wiederkehrende negative Themen werden erkannt
- **Jahresbericht**: Umfassende Analyse aller Einträge eines Jahres

### Jahresbericht erstellen

1. Navigiere zu "Jahresbericht anzeigen"
2. Wähle das gewünschte Jahr
3. Klicke auf "Bericht generieren"
4. Exportiere als Markdown-Datei

## Projektstruktur

```
Reflexion/
├── src/                          # React Frontend
│   ├── components/
│   │   ├── Dashboard/           # Hauptansicht
│   │   ├── InputSession/        # Reflexions-Eingabe
│   │   ├── YearReport/          # Jahresbericht
│   │   └── shared/              # Wiederverwendbare Komponenten
│   ├── stores/                  # Zustand State Management
│   ├── types/                   # TypeScript Definitionen
│   ├── styles/                  # Tailwind CSS
│   ├── App.tsx                  # Router & Layout
│   └── main.tsx                 # Entry Point
├── src-tauri/                   # Rust Backend
│   ├── src/
│   │   ├── main.rs             # Tauri Entry Point
│   │   ├── lib.rs              # App Setup
│   │   ├── database.rs         # SQLite Operations
│   │   ├── llm.rs              # Claude API Integration
│   │   └── commands.rs         # Tauri Commands
│   ├── Cargo.toml              # Rust Dependencies
│   └── tauri.conf.json         # Tauri Config
├── .env.example                # Environment Template
├── package.json                # Node Dependencies
└── README.md
```

## Technologie-Stack

### Frontend
- **React 19** mit TypeScript
- **Tailwind CSS** für Styling
- **React Router DOM** für Navigation
- **Zustand** für State Management
- **react-markdown** für Bericht-Rendering
- **Vite** als Build-Tool

### Backend
- **Tauri 2.x** - Desktop-Framework
- **Rust** - Backend-Logik
- **SQLite** (rusqlite) - Lokale Datenbank
- **reqwest** - HTTP Client für API-Calls
- **chrono** - Datums-Handling
- **serde** - Serialisierung

### LLM Integration
- **Claude API** (Anthropic)
- Model: `claude-sonnet-4-5-20250929`
- Features: Zusammenfassungen, Kategorisierung, Jahresberichte

## Datenbank

Die SQLite-Datenbank wird automatisch beim ersten Start erstellt.

**Speicherort:**
- **macOS**: `~/Library/Application Support/com.reflexion.app/reflexion.db`
- **Linux**: `~/.local/share/reflexion-app/reflexion.db`
- **Windows**: `%APPDATA%\reflexion-app\reflexion.db`

### Schema

- `entries`: Wöchentliche Reflexionen
- `categories`: Extrahierte Kategorien
- `follow_ups`: Wiederkehrende Themen
- `streaks`: Streak-Tracking
- `settings`: App-Konfiguration

## Tastatur-Shortcuts

- **Enter/Tab**: Weiter zum nächsten Schritt
- **Escape**: Zurück zum vorherigen Schritt
- **Cmd/Ctrl + Enter**: Formular abschließen (in Textfeldern)

## Troubleshooting

### LLM-Funktionen funktionieren nicht

1. Überprüfe, ob `.env` Datei existiert und `LLM_API_KEY` gesetzt ist
2. Teste den API-Schlüssel in der Console: https://console.anthropic.com/
3. Überprüfe die Logs in der Developer Console (Cmd/Ctrl + Shift + I)

### App startet nicht

1. Stelle sicher, dass alle Voraussetzungen installiert sind
2. Führe `npm install` erneut aus
3. Überprüfe Rust-Installation: `rustc --version`
4. Siehe Tauri-Logs in der Konsole

### Datenbank-Fehler

Die Datenbank wird automatisch initialisiert. Bei Problemen:
1. Schließe die App
2. Lösche die Datenbank-Datei (siehe Speicherort oben)
3. Starte die App neu

## Sicherheit & Datenschutz

- **Lokale Datenhaltung**: Alle Reflexionen werden lokal in SQLite gespeichert
- **Keine Cloud-Sync**: Deine Daten bleiben auf deinem Gerät
- **API-Schlüssel**: Wird nur für LLM-Anfragen verwendet, nie gespeichert oder geteilt
- **Open Source**: Der gesamte Code ist einsehbar

## Backup

Um deine Reflexionen zu sichern:

1. Finde die SQLite-Datei (siehe Speicherort oben)
2. Kopiere `reflexion.db` an einen sicheren Ort
3. Optional: Exportiere Jahresberichte als Markdown

## Lizenz

MIT

## Credits

Entwickelt mit:
- [Tauri](https://tauri.app/)
- [React](https://react.dev/)
- [Anthropic Claude](https://www.anthropic.com/)
- [Tailwind CSS](https://tailwindcss.com/)

---

**Viel Erfolg bei deiner Selbstreflexion! 🌱**
