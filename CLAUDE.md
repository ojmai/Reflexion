# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reflexion is a single-file HTML app for weekly self-reflection. No build step, no dependencies, no server — just open `index.html` in a browser. Data is stored in `localStorage`. LLM features (summaries, follow-up questions, yearly reports) call Claude or OpenAI directly from the browser.

## Running the App

Open `index.html` directly in a browser. That's it.

## Architecture

Everything lives in `index.html`: inline CSS, then a `<script>` block with all logic. No framework, no modules.

### Code structure inside `<script>`

| Section | What it does |
|---|---|
| **STORAGE** | `getEntries/saveEntries`, `getStreak/saveStreak`, `getClaudeKey`, `getOpenAIKey`, `hasApiKey()`, `getActiveProvider()` |
| **HELPERS** | ISO week/year calc, `formatDate`, `esc()` (HTML escaping), streak logic, recurring negative theme detection |
| **LLM** | `callLLM(prompt, maxTokens)` routes to Claude or OpenAI. `analyzeEntry`, `generateFollowUpQuestions`, `generateYearReport` build prompts and parse JSON responses |
| **STATE** | Single `state` object, mutated directly, re-rendered via `render()` |
| **NAVIGATION** | `navigate(view)` switches top-level views. Input view also runs async follow-up question generation on load |
| **INPUT SESSION** | `getInputSteps()` builds the conditional step list. `inputNext/inputBack/submitEntry` drive the flow |
| **RENDER** | `render()` calls one of four view functions, sets `app.innerHTML`, then `afterRender()` wires up textarea listeners |
| **SETTINGS** | Save/clear functions for both API keys |

### Views
- `dashboard` — streak, last 8 entries, start button
- `input` — multi-step flow: `greeting → [review] → [followup] → good → bad → summary → processing`
- `report` — year selector, generate via LLM, markdown rendered with `marked.js` (CDN), export as `.md`
- `settings` — Claude + OpenAI key inputs, data export/delete

### Data model (localStorage keys)
- `reflexion_entries` — `Array<{id, created_at, week_number, year, good_text, bad_text, summary, categories: [{name, sentiment}]}>`
- `reflexion_streak` — `{current_streak, longest_streak, last_entry_date}`
- `reflexion_api_key` — Claude API key
- `reflexion_openai_key` — OpenAI API key

### LLM provider logic
- Claude is used if a Claude key is set, regardless of whether an OpenAI key is also present
- OpenAI (`gpt-4o`) is used only when no Claude key is set
- `hasApiKey()` returns true if either key is set; use this for all "is LLM configured?" checks (not `getClaudeKey()`)

### Streak logic
- `< 7 days` since last entry: no change (same week)
- `7–14 days`: increment streak
- `> 14 days`: reset to 1

### Prompts
All prompts are in German. `analyzeEntry` and `generateFollowUpQuestions` expect JSON responses; `extractJson()` strips markdown code fences before parsing. `generateYearReport` returns freeform Markdown.

### HTML escaping
Always use `esc()` when inserting user data into HTML strings. The function escapes `&`, `<`, `>`, `"`. Textarea content in particular must be escaped since entity references are decoded inside `<textarea>`.
