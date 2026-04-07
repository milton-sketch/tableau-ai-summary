# 🤖 AI Summary — Tableau Extension
> AI-powered dashboard narratives using Google Gemini, delivered directly inside Tableau.

---

## What

A custom Tableau Dashboard Extension that reads live data from your Tableau worksheets and generates a natural language AI summary at the top of your dashboard. The summary is produced by Google Gemini and updates on demand with the click of a button.

**Key capabilities:**
- Reads live, filtered data directly from any worksheet in the dashboard
- Supports single-sheet or full dashboard (all sheets combined) analysis
- Displays active filters as context pills alongside the summary
- Fully customizable prompt — no code changes needed
- Works on both Tableau Desktop and Tableau Cloud
- Model selector dynamically loads available Gemini models from your API key

---

## Why

Tableau is excellent at visualizing data, but it doesn't natively explain what the data means in plain language. Executives and non-technical stakeholders often need a written narrative to complement the charts — answering "so what?" rather than leaving interpretation entirely to the viewer.

**Problems this solves:**
- Reduces the time analysts spend writing manual commentary on reports
- Makes dashboards more accessible to non-technical audiences
- Provides consistent, on-demand insights that reflect the current filter state
- Eliminates the need for expensive third-party NLG (Natural Language Generation) tools

**Why build it instead of buying it?**
- No existing free Tableau extension does this with a customizable LLM prompt
- Commercial alternatives (Narrative Science, Einstein Discovery) require significant additional licensing
- This approach gives full control over the prompt, tone, and model
- Data flows directly from Tableau to Gemini — no third-party intermediary

---

## How

### Architecture

```
Tableau Dashboard
  └── Extension Panel (GitHub Pages — index.html)
        ├── Tableau Extensions API  →  reads worksheet data & filters
        ├── Google Gemini API       →  generates the AI narrative
        └── Renders summary inside the dashboard panel
```

### Technology Stack

| Component | Technology |
|---|---|
| Extension UI | HTML, CSS, Vanilla JavaScript |
| Tableau integration | Tableau Extensions API (`tableau.extensions.1.latest.min.js`) |
| AI model | Google Gemini (configurable — Flash, Pro, etc.) |
| Hosting | GitHub Pages (free, HTTPS) |
| API key storage | Browser `sessionStorage` (cleared on tab close) |

### How It Works — Step by Step

1. **Initialization** — On load, the extension calls `tableau.extensions.initializeAsync()` to connect to the host dashboard and retrieve the list of available worksheets.
2. **Data reading** — When "Generate Summary" is clicked, the extension calls `sheet.getSummaryDataAsync()` (up to 10,000 rows) and `sheet.getFiltersAsync()` to capture the current state of the dashboard including all active filters.
3. **Prompt construction** — The data is serialized into a structured text block and injected into a prompt template using variables (`{{sheetName}}`, `{{filters}}`, `{{data}}`, `{{rowCount}}`).
4. **AI generation** — The prompt is sent to the Gemini API (`/v1beta/models/{model}:generateContent`). The response is extracted and displayed in the summary panel.
5. **Dashboard mode** — When "Entire dashboard" mode is selected, all sheets are read in parallel using `Promise.all()` and combined into a single prompt, asking Gemini to synthesize cross-sheet insights.

### Prompt System

The extension supports four tone modes:

| Tone | Description |
|---|---|
| Executive | 3–4 sentences, business-focused, no jargon |
| Analyst | 4–5 sentences, data-driven, includes specific numbers |
| Friendly | 3–4 sentences, plain English for any audience |
| Custom | User-defined prompt with live variable injection |

Custom prompts support these variables:

| Variable | Value injected |
|---|---|
| `{{sheetName}}` | Name of the worksheet being summarised |
| `{{filters}}` | All active filters currently applied |
| `{{data}}` | Serialized row data from the worksheet |
| `{{rowCount}}` | Total number of rows in the dataset |

---

## Where

### Files & Hosting

| File | Location | Purpose |
|---|---|---|
| `index.html` | GitHub Pages (`milton-sketch.github.io/tableau-ai-summary/`) | The extension UI and all logic |
| `tableau.extensions.1.latest.min.js` | Same GitHub Pages repo | Tableau Extensions API library (served locally to avoid CDN issues in Desktop) |
| `ai-summary.trex` | Local / shared with team | Tableau manifest file pointing to the hosted extension URL |

### Tableau Deployment

| Environment | Status | Notes |
|---|---|---|
| Tableau Desktop | ✅ Working | Load via Dashboard → Extension → Access Local Extensions → select `.trex` file |
| Tableau Cloud | ✅ Working | Extension URL must be added to the site safe list by a Site Administrator |

### Tableau Cloud Safe List
The following URL has been approved on the Tableau Cloud site under **Settings → Extensions → Safe list**:
```
https://milton-sketch.github.io/tableau-ai-summary/index.html
```
Permissions granted: Run extension + Access underlying data.

---

## When

### Project Timeline

| Date | Milestone |
|---|---|
| April 2026 | Concept — evaluated native Tableau AI options (Pulse, Einstein) and determined a custom extension was needed |
| April 2026 | Built MVP — single sheet summary with Anthropic Claude API |
| April 2026 | Switched to Google Gemini API (free tier, broader model availability) |
| April 2026 | Resolved Tableau Desktop compatibility (local JS library, manifest fixes) |
| April 2026 | Added dynamic model selector, custom prompt editor, variable chips |
| April 2026 | Added "Entire dashboard" mode — multi-sheet parallel analysis |
| April 2026 | Increased row limit to 10,000 (Tableau API maximum) |
| April 2026 | Published to Tableau Cloud — added to site safe list |
| April 2026 | ✅ POC complete — shared with team |

### When to Use This Extension

- At the **top of any executive dashboard** to provide immediate context
- When **sharing reports with stakeholders** who may not interpret charts independently
- After **applying filters** to get an AI-generated explanation of the filtered view
- During **weekly/monthly reviews** as a starting point for discussion

---

## Setup Guide (for new users)

### Prerequisites
- Tableau Desktop 2021.1+ or Tableau Cloud access
- A Google Gemini API key (free at [aistudio.google.com](https://aistudio.google.com))
- The `.trex` manifest file (request from the team)

### Steps

1. **In Tableau Desktop** — open a dashboard, drag an **Extension** object onto it, click "Access Local Extensions", and select `ai-summary.trex`
2. **In Tableau Cloud** — the extension loads automatically if the workbook was published with it (URL is already on the safe list)
3. **First-time setup** — on the setup screen:
   - Enter your Gemini API key
   - Click **Load Models** and select a model (Gemini Flash recommended)
   - Choose analysis mode: Single sheet or Entire dashboard
   - Select your preferred tone
   - Optionally write a custom prompt in the Prompt tab
   - Click **Save & Continue**
4. **Generate a summary** — click **✨ Generate Summary** at any time

### API Key Notes
- Each user needs their own Gemini API key
- The key is stored in `sessionStorage` — it is never sent to any server other than Google's Gemini API
- The key is cleared when the browser tab or Tableau session is closed — users will need to re-enter it each session (by design, for security)

---

## Limitations & Known Constraints

| Limitation | Detail |
|---|---|
| Max rows | 10,000 rows per sheet (Tableau API hard limit on `getSummaryDataAsync`) |
| API key per session | Key must be re-entered each Tableau session (sessionStorage) |
| No key sharing | Each user provides their own Gemini key — no shared backend |
| Network dependency | Requires internet access to reach the Gemini API |
| Sandboxed extensions | Tableau's sandboxed extension mode blocks external API calls — non-sandboxed mode required |

---

## Potential Next Steps

- **Backend proxy** — move the API key to a server so users don't need their own Gemini key
- **Auto-refresh** — regenerate summary automatically when dashboard filters change
- **Summary history** — store previous summaries per dashboard for comparison
- **Multi-language** — add a language selector to generate summaries in Spanish, French, etc.
- **Export** — add a button to copy or download the summary as plain text
- **Anthropic Claude support** — re-add as an alternative AI provider option

---

*Documentation generated April 2026 | Extension version 2.2 | Built with Tableau Extensions API + Google Gemini*