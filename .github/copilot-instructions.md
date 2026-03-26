# Copilot Instructions — AMD Template Tools

## Project Overview

**AMD Template Tools** is a suite of client-side web tools for **TMJ Sleep Solutions NW** that enables staff to create templates for **AdvancedMD (AMD)**, a healthcare EHR system. There are two tools:

- **Email Template Designer** (`amd-email-designer.html`) — Build visual responsive email templates for patient messaging (appointment reminders, telehealth links, etc.) that are pasted into AMD's email editor.
- **In-Line Narrative Template Designer** (`amd-note-builder.html`) — Create structured clinical note templates for AMD's narrative template editor used in patient charts.

Both tools integrate with AMD's **smart fields** (bookmarks) which auto-populate from patient chart data at runtime.

## Architecture & Tech Stack

- **Pure vanilla JS/HTML/CSS** — no frameworks, no build tools, no npm, no external dependencies.
- Each tool is a **single self-contained HTML file** with all CSS and JavaScript inlined.
- **No build process** — edit and reload. Files are served as static HTML.
- **Client-side persistence** via `localStorage` (autosave, snippets, last-used tool).
- **No server required** — can be run locally or on any static host.

## File Structure

```
index.html                    # Landing page / navigation hub
amd-email-designer.html       # Email template builder (~144 KB)
amd-note-builder.html         # Clinical note builder (~135 KB)
README.md                     # Minimal placeholder
```

## Key Conventions

### State Management
- Global settings object `G` holds template-wide settings (fonts, colors, padding).
- `sections` array holds all blocks/sections as plain JS objects.
- History stack (max 100 entries) powers undo/redo via `pushHist()`.

### Section/Block Shape
```js
{
  id: "_abc12345",   // unique ID from uid()
  type: "header",    // section type string
  text: "...",       // content (varies by type)
  bg: "#ffffff",     // background color
  // ...type-specific props
}
```

### Naming Conventions
- Utility: `uid()`, `clone()`, `S()` (escape), `SF()` (smart-field format)
- UI: `doAdd()`, `delSection()`, `duplicateSection()`, `moveSection()`
- Rendering: `renderCanvas()`, `renderSection()`, `renderButtons()`
- Storage: `autosave()`, `loadSavedSession()`, `clearSavedSession()`
- History: `pushHist()`, `undo()`, `redo()`, `updateUR()`

### CSS Conventions
- All colors are CSS custom properties (`--blue`, `--text`, `--surface`, `--border`, etc.)
- Section wrappers use `.canvas-section`, `.sec-inner`, `.sec-action-bar`
- Floating panels use `.fp-*` prefix; splash screen uses `.splash-*`
- State classes: `.active` (selected), `.hidden-sec` (hidden), `.mf-flash` (bookmark inserted)

### localStorage Keys
| Key | Tool | Contents |
|-----|------|----------|
| `td_autosave` | Email Designer | `{ sections, G, savedAt }` |
| `td_snippets` | Email Designer | Array of section objects |
| `nb_autosave` | Note Builder | `{ sections, G, savedAt }` |
| `nb_snippets` | Note Builder | Array of block objects |
| `amt_last_tool` | index.html | Last used tool identifier |

## Domain Context

- **Healthcare / Sleep Medicine**: Templates cover workflows like HST (Home Sleep Test), OSD (Oral Sleep Device) fittings, CPAP/DME setup, telehealth visits, follow-up appointments.
- **AMD Bookmarks**: Smart fields are AMD-specific tokens (e.g., `[PATIENT_FIRST_NAME]`, `[PROVIDER_FULL_NAME]`) rendered as styled non-editable `<span>` elements on the canvas and exported with `data-bookmark-additional` class.
- **Export target**: HTML is copied to clipboard for direct pasting into AdvancedMD's rich-text editor.

## Section Types

**Email Designer (8 types):** Header, Paragraph, Button, Notice, Divider, Spacer, Footer, Columns, Image

**Note Builder (6 types):** Heading (h3/h4/h5), Paragraph, List (ul/ol), Table, Divider, Spacer

## Development Guidelines

- **Do not introduce external libraries or build steps.** Keep everything self-contained in the HTML files.
- **Do not add a package.json, node_modules, or bundler config** unless the task explicitly requires a new tool.
- Preserve the single-file pattern — CSS and JS stay inlined in each HTML file.
- Follow existing naming conventions and CSS variable usage.
- All color values should use the defined CSS custom properties, not hardcoded hex.
- When adding new section types, follow the existing pattern: add to `doAdd()`, `renderSection()`, and the splash/template system.
- Smart field spans must remain `contenteditable="false"` and carry the `amds-narrative-template-field` class for AMD compatibility.
