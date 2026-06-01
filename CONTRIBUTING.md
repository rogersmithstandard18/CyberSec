# Contributing to CyberSec

Thank you for your interest in contributing. This document covers how to add or update content, the documentation standards this repo follows, and how changes move from draft to merge.

---

## Table of Contents

- [Project philosophy](#project-philosophy)
- [What we welcome](#what-we-welcome)
- [Documentation standards](#documentation-standards)
- [Code standards](#code-standards)
- [Adding a new topic to Topicgen](#adding-a-new-topic-to-topicgen)
- [Adding a new incident to SOC-Simulator](#adding-a-new-incident-to-soc-simulator)
- [Adding a new role or altitude view](#adding-a-new-role-or-altitude-view)
- [Submitting changes](#submitting-changes)
- [Style guide](#style-guide)

---

## Project philosophy

Both tools in this repo prioritize **accuracy over completeness**. A smaller set of well-documented, technically correct scenarios is more valuable than a large set of vague ones. Before adding content, ask: *Would a working SOC analyst recognize this as realistic?*

---

## What we welcome

- New incident scenarios in `data.js` with complete altitude views
- Additional MITRE ATT&CK–mapped topics in Topicgen
- Documentation improvements — clarity fixes, missing descriptions, broken links
- Bug fixes for UI rendering issues
- New roles, provided they represent a real org structure position
- README updates that improve setup or usage clarity

**Not currently in scope:**
- Backend integrations or server-side dependencies
- Authentication or user account features
- Content that requires a build step to run

---

## Documentation standards

All documentation in this repo follows these standards. Apply them when writing new content and when editing existing content.

### READMEs

Every project directory must contain a `README.md` that covers:

1. **What it is** — one paragraph, plain language, no assumed context
2. **How to run** — exact steps from a cold start, including any file to open
3. **Key concepts** — definitions for any domain-specific terms used in the UI
4. **File structure** — annotated tree of all non-trivial files
5. **Customization** — any settings or variables a user might want to change

READMEs are written for two audiences simultaneously: a cybersecurity practitioner who has never used this tool, and a technical recruiter who may not be a practitioner. Avoid acronyms without definition on first use.

### In-line code comments

All source files should include:

- A **file header comment** (first three lines): tool name, file name/role, and one-sentence summary of what the file is responsible for.
- **Section dividers** for logical blocks within a file, using the `// ─── SECTION NAME ──` style already established in `data.js`.
- **Prop and parameter comments** for any function whose inputs are not self-evident from names alone.
- **"Why not what" comments** for non-obvious logic: explain the reason behind a decision, not what the code is doing (the code already shows that).

Example of a good comment:
```js
// Role altitude drives nav visibility — higher altitude = fewer operational pages,
// more strategic ones. This keeps T1 analysts from seeing board report drafts.
const navIds = window.NAV_BY_ALTITUDE[altitude] || ["home"];
```

Example of a comment that adds no value (avoid):
```js
// Get the nav IDs for this altitude
const navIds = window.NAV_BY_ALTITUDE[altitude] || ["home"];
```

### Tables

Use Markdown tables for any list of items that have two or more attributes. Prefer tables over bulleted lists for reference content. Column headers should be title case. Keep cell content concise — link out rather than embedding long descriptions.

### Tone and voice

- Write in second person ("you") for instructions and tutorials.
- Write in third person for reference content (API docs, data schemas, component descriptions).
- Use active voice. *"The role switcher filters the nav"* not *"The nav is filtered by the role switcher."*
- Define abbreviations on first use: *"MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge)"*.
- Avoid filler phrases: *"simply," "just," "easily," "of course."*

---

## Code standards

### File headers

Every `.jsx` and `.js` file must start with a three-line header comment:

```js
// Stratum SecOps — [filename]
// [One sentence: what this file owns / is responsible for.]
// [Optional: one sentence on any non-obvious architectural decision.]
```

### Globals

All shared state is exposed on `window` via `Object.assign(window, { ... })` at the bottom of each file. Do not introduce module imports — the tool runs without a build step and relies on global script order. New globals should be documented in the **Globals reference** section of `SOC-Simulator/README.md`.

### Naming

| Type | Convention | Example |
|---|---|---|
| React components | PascalCase | `AltitudeRow` |
| Window globals | SCREAMING_SNAKE | `ALTITUDE_VIEWS` |
| Local variables | camelCase | `navIds` |
| CSS custom properties | `--kebab-case` | `--bg-elev` |
| File names | kebab-case for assets | `styles.css` |

### CSS

All colors, spacing, and typography must reference the custom properties defined in `styles.css`. Do not hardcode hex values in component files except in `data.js` where colors are part of the data model (department colors, severity colors).

---

## Adding a new topic to Topicgen

Topics live in the `topicsDatabase` array inside `Topicgen`. Each entry follows this schema:

```js
{
  id: string,          // MITRE technique ID (e.g. "T1059.001") or custom ID for defensive topics
  title: string,       // Short name, title case, max ~50 chars
  description: string, // One to two sentences. Plain language. What would you do / detect?
  type: string,        // "Offensive" or "Defensive"
  mitreId: string,     // MITRE ATT&CK or D3FEND ID for reference
  difficulty: string,  // "Beginner", "Intermediate", or "Advanced"
}
```

**Guidelines for topic content:**

- `description` should describe a concrete action or scenario, not a definition. *"Inject malicious code into legitimate processes to evade detection"* is better than *"A technique involving process injection."*
- Pair offensive and defensive topics where possible. If you add `T1059.001` (PowerShell execution), consider also adding a detection/mitigation counterpart.
- Difficulty ratings should be consistent with MITRE's published guidance and community consensus. When uncertain, default to `"Intermediate"`.

---

## Adding a new incident to SOC-Simulator

Incidents live in `window.INCIDENTS` in `src/data.js`. To be complete, a new incident requires:

1. **An entry in `INCIDENTS`** — all required fields (see the INC-9201 entry as the canonical example).
2. **An entry in `ALTITUDE_VIEWS`** keyed to the same incident ID — one view object per role ID, covering every role in the `order` array in `altitude.jsx`.
3. **A timeline** — at minimum five events, with realistic timestamps and named responders drawn from the existing `ROLES` list.

An incident without a complete altitude view will not render in the Altitude View page and will show a fallback error. Do not merge incomplete incidents.

**Required fields for `INCIDENTS`:**

| Field | Type | Description |
|---|---|---|
| `id` | string | Format: `INC-NNNN`. Increment from the highest existing ID. |
| `title` | string | Plain-language description. Max ~80 chars. |
| `kind` | string | Must match a key in `KIND_META` (e.g. `"c2"`, `"fraud"`, `"ad"`). |
| `severity` | string | `"Critical"`, `"High"`, `"Medium"`, or `"Low"`. |
| `status` | string | Must match a key in `STATUS_META`. |
| `dept` | string | Must match a key in `DEPARTMENTS`. |
| `mitre` | string | One or more MITRE ATT&CK technique IDs, dot-separated. |
| `summary` | string | Two to three sentences. Technical facts only — no editorial framing. |
| `timeline` | array | Array of `{ t, who, what }` objects. `t` is a UTC timestamp string. |

---

## Adding a new role or altitude view

Roles are defined in `window.ROLES` in `data.js`. Each role has an `altitude` value (0–6) that controls which pages appear in that role's sidebar nav, as defined by `NAV_BY_ALTITUDE`.

Before adding a role:

- Confirm the role represents a distinct organizational position with a meaningfully different information need than existing roles at the same altitude.
- Update `NAV_BY_ALTITUDE` if the new role requires a different page set.
- Add altitude view entries in `ALTITUDE_VIEWS` for all existing incidents that have complete altitude views. A new role without altitude views will be silently omitted from the Altitude View page.

---

## Submitting changes

1. Fork the repository and create a branch: `git checkout -b feature/your-description`
2. Make your changes. Run the affected HTML file in a browser to verify.
3. Update documentation: if you changed a data schema, update the relevant README. If you changed a component's props, update its file header comment.
4. Open a pull request with a clear title and a short description of what changed and why.

There are no automated tests. Manual verification in a modern browser (Chrome, Firefox, or Safari) is required before submitting.

---

## Style guide

This repo follows a minimal house style based on [Google Developer Documentation Style Guide](https://developers.google.com/style) with the following overrides:

| Rule | This repo's preference |
|---|---|
| Headings | Sentence case (`## File structure`, not `## File Structure`) |
| Code in prose | Always use backticks for filenames, commands, variable names, and values |
| Lists | Use unordered lists for items without rank or sequence; ordered lists for steps |
| Numbers | Spell out zero through nine; use numerals for 10 and above |
| Em dashes | Use `—` (no spaces) for parenthetical asides in prose |
| Contractions | Permitted in instructional content; avoid in reference content |

When in doubt, optimize for the reader who is encountering this project for the first time.
