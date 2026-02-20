# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reflexion is a native desktop app for weekly self-reflection built with Tauri 2.x, React 19, TypeScript, and SQLite. The app features LLM-powered summaries using Claude API for automatic categorization and yearly report generation.

## Development Commands

### Running the App
```bash
npm run tauri dev           # Start development mode (Vite dev server + Tauri app)
npm run build               # Build frontend only (TypeScript + Vite)
npm run tauri build         # Build production app bundle
```

### Frontend Only
```bash
npm run dev                 # Start Vite dev server only (port 1420)
npm run preview             # Preview production build
```

### LLM Configuration
The app loads `.env` from two locations (in order):
1. Project directory: `/path/to/Reflexion/.env` (development)
2. Home directory: `~/.reflexion.env` (production app)

```bash
cp .env.example .env              # Development setup
cp .env ~/.reflexion.env          # Production app setup
# Edit and add: LLM_API_KEY=sk-ant-xxxxx
```

**Important:** Production apps built with `npm run tauri build` must use `~/.reflexion.env` since they don't run from the project directory.

## Architecture Overview

### Tauri Bridge Pattern
This is a **hybrid architecture** where React frontend communicates with Rust backend via Tauri commands:

**Data Flow:**
1. React components call Tauri commands via `invoke()` from `@tauri-apps/api/core`
2. Commands defined in `src-tauri/src/commands.rs` handle business logic
3. Commands access shared `AppState` (Database + LLMConfig) via `State<'_, AppState>`
4. Database operations in `database.rs` manage SQLite
5. LLM operations in `llm.rs` handle Claude API calls
6. Results serialize back to TypeScript types

**Key Invariant:** All Rust structs used in commands MUST derive `Serialize`/`Deserialize` and have matching TypeScript interfaces in `src/types/index.ts`.

### State Management Architecture

**Rust State (Backend):**
- `AppState` in `lib.rs` contains:
  - `db: Mutex<Database>` - Thread-safe SQLite connection
  - `llm_config: LlmConfig` - Loaded from .env on startup via `dotenvy::dotenv()`
- Shared across all Tauri commands via Tauri's managed state

**React State (Frontend):**
- Zustand store in `src/stores/appStore.ts` manages:
  - `currentEntry`, `streak`, `followUps`, `llmConfigured`
  - Actions that invoke Tauri commands and update local state
- Components consume store via `useAppStore()` hook

### Database Schema & Auto-initialization

SQLite database at `~/Library/Application Support/com.reflexion.app/reflexion.db` (macOS).

**Schema automatically created on first run by `Database::init_schema()`:**
- `entries` - Weekly reflections (good_text, bad_text, summary, processed flag)
- `categories` - LLM-extracted categories with sentiment (positive/negative)
- `follow_ups` - Recurring negative themes tracked across weeks
- `streaks` - Single row (id=1) tracking current/longest streak + last_entry_date
- `settings` - Key-value config store

**Streak Logic:** `update_streak()` calculates:
- Same week (< 7 days): No change
- 7-14 days: Increment streak
- > 14 days: Reset to 1

### LLM Integration Pattern

**Three Claude API Functions (all async):**
1. `analyze_entry()` - Returns `CategoryAnalysis { summary, categories }`
2. `generate_follow_up_questions()` - Returns `Vec<String>` of questions
3. `generate_year_report()` - Returns Markdown string

**Prompt Strategy:**
- All prompts are in **German** (as specified in requirements)
- Responses expected in JSON format (extracted from potential markdown code blocks)
- Model: `claude-sonnet-4-5-20250929`
- Endpoint: `https://api.anthropic.com/v1/messages`

**Configuration via .env:**
```
LLM_PROVIDER=claude              # Default, supports openai/ollama
LLM_API_KEY=sk-ant-xxxxx         # Required for LLM features
LLM_MODEL=claude-sonnet-4-5-20250929  # Optional override
LLM_BASE_URL=...                 # Optional for self-hosted
```

### Component Architecture

**Route Structure (React Router):**
- `/` - Dashboard (overview, streak, recent entries)
- `/input` - InputSession (7-step reflection flow)
- `/report/:year` - YearReport (generate & export yearly reports)

**InputSession Flow State Machine:**
```
greeting → review → followup → good → bad → summary → processing
```
Steps are conditional:
- Skip `review` if no lastEntry
- Skip `followup` if no questions/follow-ups
- Keyboard navigation: Escape = back, Enter = next

**Styling System:**
- Tailwind v4 with `@import "tailwindcss"` in `globals.css`
- **IMPORTANT:** Due to Tailwind v4 limitations, button styles use **inline Tailwind classes** directly in components, NOT custom CSS classes
- Button pattern (used throughout):
  ```tsx
  // Primary button
  className="px-6 py-3 bg-gradient-to-br from-blue-500 to-blue-700 hover:from-blue-600 hover:to-blue-800 text-white font-semibold rounded-xl shadow-lg transition-all duration-200 hover:shadow-xl hover:-translate-y-0.5 active:translate-y-0 active:scale-98"

  // Secondary button
  className="px-6 py-3 bg-zinc-800 hover:bg-zinc-700 text-white font-semibold rounded-xl border border-zinc-700 transition-all duration-200 hover:shadow-lg hover:-translate-y-0.5 active:translate-y-0 active:scale-98"
  ```
- Custom utility classes in `globals.css`:
  - `.card`, `.card-hover` (glassmorphism with backdrop-blur)
  - `.input`, `.textarea` (form inputs with focus states)
  - `.text-text-secondary`, `.accent-green`, etc.

## Adding New Features

### Adding a New Tauri Command
1. Define command function in `src-tauri/src/commands.rs` with `#[tauri::command]`
2. Add to `invoke_handler` list in `src-tauri/src/lib.rs`
3. Create matching TypeScript types in `src/types/index.ts`
4. Call via `invoke<ReturnType>('command_name', { args })` in React

### Adding Database Tables/Columns
1. Update schema in `Database::init_schema()` in `database.rs`
2. Add `CREATE TABLE IF NOT EXISTS` - safe for existing DBs
3. Add corresponding struct with `#[derive(Serialize, Deserialize)]`
4. Add CRUD methods to `impl Database`
5. Expose via Tauri commands if frontend needs access

### Modifying LLM Prompts
Prompts in `src-tauri/src/llm.rs` are German-language strings with JSON response format.
Key considerations:
- Keep JSON schema clear in prompt
- Handle markdown code block wrapping (`\`\`\`json`)
- Use `serde_json::from_str()` for parsing
- Return `anyhow::Result` for error propagation

## Common Issues

### TypeScript Type Mismatches
If Tauri command returns data that doesn't match TypeScript interface:
1. Check Rust struct has correct `#[derive(Serialize)]`
2. Verify field names match exactly (Rust snake_case = TypeScript camelCase via serde)
3. Check `Option<T>` in Rust = `T | undefined` in TypeScript

### LLM Features Not Working
1. Verify `.env` exists with `LLM_API_KEY`
2. Check `llmConfigured` state in Dashboard warning banner
3. App loads .env on startup - restart after changes
4. Check browser dev console for error messages from failed `invoke()` calls

### Styling Not Applied
- Tailwind v4 doesn't support `@apply` with custom classes in `@layer components`
- Use plain CSS for utility classes in `globals.css`
- **For buttons:** Use inline Tailwind classes (see Styling System section above) - do NOT create custom `.btn` classes as they won't override Tailwind properly
- Standard Tailwind utilities (e.g., `bg-zinc-900`) work in JSX
- If custom styles don't work, add `!important` in CSS or use inline Tailwind classes

### Database Locked Errors
- `AppState.db` is `Mutex<Database>` - always `.lock()?` before use
- Never hold lock across `await` points - unlock before async LLM calls
- Pattern: Read from DB, release lock, then process data

## Database Location by OS
- macOS: `~/Library/Application Support/com.johannes.reflexion-app/reflexion.db`
- Linux: `~/.local/share/reflexion-app/reflexion.db`
- Windows: `%APPDATA%\reflexion-app\reflexion.db`

To reset: Close app, delete DB file, restart (auto-reinitializes schema).

## Production Build & Deployment

### Building the Production App
```bash
export PATH="$HOME/.cargo/bin:$PATH"  # Ensure Rust toolchain in PATH
npm run tauri build
```

Output locations:
- App bundle: `src-tauri/target/release/bundle/macos/reflexion-app.app`
- DMG installer: `src-tauri/target/release/bundle/dmg/reflexion-app_0.1.0_aarch64.dmg`

### Installing to Applications
```bash
cp -r src-tauri/target/release/bundle/macos/reflexion-app.app /Applications/Reflexion.app
```

### Window Configuration
Window settings in `src-tauri/tauri.conf.json`:
- Default size: 1200x900px
- Centered on screen
- Resizable with minimum size: 800x700px
