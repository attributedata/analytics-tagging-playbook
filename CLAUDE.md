# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

The **Analytics Tagging Playbook**: our team's internal standard for tagging websites and apps with Adobe Analytics (via Adobe Launch) and Google Analytics 4 (via Google Tag Manager). It is a docs-as-code site: Markdown files in `docs/`, built into a static website with MkDocs and the Material theme, and published through GitHub Actions.

We are a digital analytics consultancy. We work for several clients. Our usual process is:

1. We write a **technical specification** for the client's developers. It shows the exact JSON object to push to the data layer, when to fire it, and what each key holds.
2. Developers add the pushes to the site or app.
3. **We build the tagging**: data elements, rules and variable mappings in Launch or GTM. We do not write full code. We write custom code only when a rule cannot do the job.

This playbook is the team's go-to source for that work. Developers on the team (including interns) contribute through pull requests, so pages must stay easy to write and review.

The audience is **our team only**. The site is internal. It must stay **client-agnostic**: nothing in it may identify a client.

## Repository layout

```
mkdocs.yml                      Site config and the navigation menu (nav). Every page must be listed here
requirements.txt                Pinned: mkdocs==1.6.1, mkdocs-material==9.7.7
README.md                       Setup and run instructions for humans
CLAUDE.md                       This file
LICENSE                         Proprietary notice
templates/tag.md                The template every tag page starts from (NOT published)
docs/
  index.md                      Home page and the "where to start" table
  conventions.md                Placeholders, data layer rules, naming, the four implementation tabs
  contributing.md               How teammates contribute (browser edit button, new topic flow)
  tags/                         Approved and draft tags (see list below)
  stylesheets/extra.css         Header colour (#181B58), fixed header title (site name never swaps to the page title), page-meta line style. The only custom CSS
  assets/logo.png               Team logo, black on transparent. Used as the favicon
  assets/logo-white.png         Same logo recoloured white, for the navy header
.github/
  workflows/docs.yml            Builds with --strict on every PR, deploys to GitHub Pages on merge to main
  pull_request_template.md      PR checklist
  CODEOWNERS                    Requires a lead's review (team name is a placeholder: YOUR-ORG/tagging-leads)
```

### Pages in `docs/tags/`

| File | Topic |
|---|---|
| `embed-code.md` | Library Installation (Launch embed, GTM snippet, mobile SDK setup) |
| `marketing-channels.md` | Marketing channels and tracking codes |
| `page-naming.md` | Page name format and the `page_view` push |
| `site-sections.md` | Site section levels derived from the page name |
| `site-search.md` | Internal search tracking |
| `form-tracking.md` | Form start, step, complete, error. The reference example for the page style |
| `clickthrough.md` | Universal clickthrough (`universal_click`) |
| `file-downloads.md` | File downloads (`file_download`) |
| `error-tracking.md` | Site errors (`site_error`) |
| `content-pages.md` | Content metadata (`content_view`), optional topic |
| `data-layer.md` | How the data layer works and the rules for developers |
| `event-serialization.md` | Deduplicating conversions with a submission ID |
| `qa-debugging.md` | Tools and the standard QA procedure |
| `video-tracking.md` | Video start and milestones (`video_milestone`). Promoted from a proposal on 29-Oct-2026, untested |

There is no Proposals section or `docs/proposals/` folder any more (removed 29-Oct-2026). New topics go straight into `docs/tags/` with status Draft.

The menu (`nav` in `mkdocs.yml`) groups pages into sections: Start here, Setup (Library Installation, Data Layer), Analytics Config (Marketing Channels), Core tags (Page Naming through Video Tracking), Supporting config (Event Serialization), Testing (QA and Debugging). The section a page sits in does not depend on its folder: all of these pages live in `docs/tags/`. `docs/contributing.md` has a table saying which section a new page belongs in.

## Commands

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve            # live preview at http://127.0.0.1:8000
mkdocs build --strict   # MUST pass before any commit. It fails on broken links and on pages missing from nav
```

`mkdocs build` also prints a long banner about MkDocs 2.0. This is a warning about a future version, safe to ignore. Do not upgrade MkDocs or the Material theme without being asked. The versions are pinned deliberately.

Do not commit the `site/` folder (it is in `.gitignore`).

## How every tag page is structured

Every page in `docs/tags/` follows `templates/tag.md`. Keep this order and these heading names, so pages look the same and readers know where to look:

1. `# Title`, then the **page info line**: one line of metadata, followed by `{ .page-meta }` on its own line so it is styled small and grey under the title. Not a table:
   ```markdown
   Status: **Draft** · Owner: **Add name** · Last reviewed: **Add date**
   { .page-meta }
   ```
2. `## What this tracks and why`
3. `## Variables required`: two tabs, each with a `Type | How many | What each one holds` table. Each row has a count and what each variable holds:
   - `=== "Adobe Analytics"`: rows eVars, Props, Events, plus any special variable (Campaign, Page Name, Channel) and Other (if any).
   - `=== "Google Analytics 4"`: rows Custom dimensions, Events.
   Notes that apply to both platforms go after the tabs, unindented.
4. `## When to fire`
5. `## Values and naming` (only where it adds something; the key table in the developer specification often covers it)
6. `## Developer specification`: the part we hand to client developers (see below)
7. `## Tagging setup`: the part our team builds (see below)
8. `## QA checklist`: checkbox list
9. `## Notes`

### Developer specification section

- Tabs, in this exact order and with these exact labels:
  - `=== "Adobe websites (adobeDataLayer)"`
  - `=== "Google Analytics websites (dataLayer)"`
  - `=== "Mobile apps"`
- Each web tab has a full copy-paste snippet that starts with the array-creation line, then pushes ONE flat object:
  ```javascript
  window.adobeDataLayer = window.adobeDataLayer || [];
  window.adobeDataLayer.push({
    'event': 'form_start',
    'formId': '<CMS form ID>'
  });
  ```
  The Google tab is identical except `window.dataLayer`.
- After the tabs: a **key table** (`Key | What it holds | Expected values and examples`, plus a `Sent on` column where keys vary by event), then an **Example** JSON block.
- Event names and keys are identical on every platform. Only the array name changes. The one existing exception is search: the Adobe push uses `search_results` and the Google push uses GA4's recommended `view_search_results`. The page explains this in an info box. Do not add more exceptions without a strong reason.
- Where developers do nothing by default (marketing channels, page naming, site sections), the section says **"Default: nothing for developers"** and explains the optional push.

### Tagging setup section

- Tabs, in this exact order and with these exact labels (the same labels on every page make the tab choice stay selected across pages, because `content.tabs.link` is on):
  - `=== "Adobe Analytics (AppMeasurement)"`
  - `=== "Adobe Web SDK (XDM)"`
  - `=== "Adobe Mobile SDK"`
  - `=== "Google Analytics 4 (GTM)"`
- Describe configuration, not code. Use tables:
  - Adobe AppMeasurement: a data element table (type *Data Layer Computed State*, path = the data layer key) and a rule table (`Rule | Trigger | Actions`).
  - Adobe Web SDK: the XDM object, the rule, and an XDM-field-to-Analytics-variable mapping table.
  - Adobe Mobile SDK: Analytics **Processing Rules** (condition to variable). Developer code lives in the developer specification, not here.
  - Google Analytics 4: Data Layer Variables, Custom Event triggers, GA4 Event tags with parameters, and which parameters to register as custom dimensions.
- Custom code is the exception. When a snippet is unavoidable, introduce it with the label "Custom code (only if needed)" and keep it short. Say what it does.
- Note GA4 enhanced measurement conflicts (form interactions, file downloads, outbound clicks, search, video) where they would double count.

## Conventions you must follow

### Placeholders, never real numbers

Tags never state real eVar, prop or event numbers. They use capitalised placeholders so they work for any client:

- `EVAR_FORM_NAME`, `PROP_SEARCH_TERM`, `EVENT_FORM_START`, and so on.
- Pattern: `EVAR_<THING>`, `PROP_<THING>`, `EVENT_<THING>`.
- Real numbers belong in each client's own "variable map", which is outside this repo.
- The one exception is event and parameter names that are fixed by the platform or by our spec (for example `form_start`, `file_download`, `search_term`).

### Data layer rules (repeat them in specs; they prevent real bugs)

- Adobe uses `window.adobeDataLayer`. Google uses `window.dataLayer`. Mobile has no data layer; developers call the Mobile SDK (`MobileCore.track(action:data:)` on iOS, `MobileCore.trackAction` on Android) with the same keys.
- Event name goes in `event`: lower case with underscores (`form_complete`).
- Keys are camelCase and **flat** (no nested objects).
- The data layer remembers the last value of each key, so **every push sends all of its keys**, otherwise old values leak into the next event.
- Values are readable text exactly as the spec lists them (`Contact us`). Our team lower-cases and trims in Launch (data element options *Force lowercase value* and *Remove extra whitespace*).
- Multiple values in one key: comma and a space (`Supply Chain, Technology`).
- Success events (`form_complete`) fire after the server confirms (HTTP 200), never on button click.
- Never send personal data. No names, emails, ID or account numbers, phone numbers, typed text, or hashes of them.

### Naming

- Launch rules start with the trigger type: `PLR` page load, `EBR` element-based, `DCR` direct call, `ACDL` data layer event. Example: `ACDL - Form: Start by "form_start"`.
- Launch data elements: `DL - keyName` (data layer value), `Query - name` (URL parameter), `Code - name` (custom code).
- GTM: `DLV - keyName` variables, `CE - event_name` triggers, `GA4 - Event Name` tags.
- Page name format: `site:section:subsection:page`, lower case, colons between levels, hyphens inside words, under 100 characters.
- Tracking code (Adobe): `channel_source_campaign_content`, lower case, URL parameter `cid`. Google uses standard `utm_*` parameters.

### Counts must stay in sync (important)

The playbook is a reference for implementing individual tags. Nobody implements every tag, so there is **no "standard build" and no totals**. The `docs/variable-requirements.md` page was removed on 29-Oct-2026 for this reason. Do not bring back totals or a combined count.

Each tag's counts appear in two places. When you change a count on a tag page, change **both in the same commit**:

1. The page's own `## Variables required` tables (both tabs).
2. The "where to start" table in `docs/index.md` (eVars, Events, GA4 dimensions columns).

Per-topic counts (eVars / props / events / GA4 dimensions / GA4 events):

| Topic | eVars | Props | Events | GA4 dims | GA4 events |
|---|---|---|---|---|---|
| Marketing Channels | 0 (1 optional) | 0 | 0 | 0 | 0 |
| Page Naming | 1 | 0 (1 optional) | 0 | 1 | 0 |
| Site Sections | 1 | 3 | 0 | 3 | 0 |
| Site Search | 3 | 1 | 2 | 2 | 2 |
| Form Tracking | 4 | 0 | 4 | 4 | 4 |
| Universal Clickthrough | 3 | 0 | 1 | 3 | 1 |
| File Downloads | 3 | 0 | 1 | 3 | 1 |
| Error Tracking | 2 | 0 | 1 | 2 | 1 |
| Video Tracking | 2 | 0 | 5 | 2 | 1 |
| Content Page Tracking | up to 9 | 0 | 1 | up to 9 | 1 |

These counts are our recommended starting point, not Adobe or Google rules.

### Status values

Every page is marked `Draft`, `Approved` or `Deprecated`. Dates are written `DD-Mon-YYYY` (`29-Oct-2026`). Only a human lead changes a status to `Approved`. **Never mark a page Approved yourself.** All current pages are Draft.

## Content rules

- **Client-agnostic.** Never include a client's name, domain, URLs, container or property IDs, report suite names, real form names, or real value lists. Use neutral examples (`acme`, `example.com`, `General enquiry`). If asked to add something from a client document, generalise it first.
- **No personal data** in any example.
- **No invented facts about Adobe or Google.** If you are not sure of a menu name, XDM field, API or limit, say so with a hedge, as existing pages do (for example "the label may differ slightly by extension version"). `docs/conventions.md` carries a standing warning to check names against the user's own setup. Keep it.
- Write for a new colleague. Short sentences, plain words, active voice. Show a good and a bad example of values. Avoid long code. Avoid filler.
- Keep the first-person plural ("we") out of tag pages unless it is about the team's process. Prefer instructions ("Create a data element...").

## Material for MkDocs syntax used here

- Tabs: `=== "Label"` followed by an indented (4 spaces) block. Everything inside a tab must be indented, including code fences and tables.
- Admonitions: `!!! note "Title"`, `!!! warning`, `!!! danger`, `!!! tip`, `!!! info`, with the body indented by 4 spaces.
- Code fences are used with a language (`javascript`, `swift`, `kotlin`, `json`, `html`, `text`). A copy button is added automatically.
- Internal links are relative file links (`[Form Tracking](form-tracking.md)`). From `docs/tags/` to the top level use `../conventions.md`. `--strict` catches mistakes.
- Do not add MkDocs plugins or theme overrides. Keep the setup minimal so contributors do not need extra tooling. The one exception is `docs/stylesheets/extra.css` (header colour and the `.page-meta` line). Keep it small.
- Block attributes use `attr_list`: put `{ .class }` on its own line directly after a paragraph, as the page info line does.

## Adding a new page (checklist)

1. Copy `templates/tag.md` into `docs/tags/<name>.md`. File names are lower case with hyphens.
2. Fill in every section from the template. Delete sections you cannot fill, but keep **Variables required** and **Developer specification**.
3. Add the page to `nav` in `mkdocs.yml`.
4. Add a row for it to the "where to start" table in `docs/index.md`, with the same counts as its Variables required section.
5. Run `mkdocs build --strict`. Fix every error.
6. Open `mkdocs serve` and look at the page, especially tabs, tables and code blocks inside tabs.

## Changing an existing spec

If a data layer key or event name changes, it changes everywhere: the key table, the example JSON, all web snippets and the mobile snippets, every tagging tab, the QA checklist, and `docs/tags/data-layer.md` (it lists the standard events). Search the whole `docs/` folder for the old name before finishing:

```bash
grep -rn "oldKeyName" docs templates
```

Add a one-line note about the change, with date, in the page's Notes section.

## CI and publishing

- `.github/workflows/docs.yml` runs `mkdocs build --strict` on every pull request and on pushes to `main`.
- On push to `main`, it runs `mkdocs gh-deploy --force`, which publishes to the `gh-pages` branch for GitHub Pages.
- The site is meant for the team only. A normal GitHub Pages site is public even for a private repo, so hosting needs to be team-only (GitHub Enterprise Cloud private Pages, or the built `site/` folder behind a company login). See `README.md`. Do not change the hosting setup without being asked.
- Repository: https://github.com/attributedata/analytics-tagging-playbook (`repo_url` in `mkdocs.yml` and the git remote `origin`).
- Placeholder still to replace by a human: the `YOUR-ORG/tagging-leads` team in `.github/CODEOWNERS`.

## Do and don't

Do:
- Run `mkdocs build --strict` before saying you are done.
- Keep pages consistent with `docs/tags/form-tracking.md`, the reference example.
- Keep tab labels and order identical on every page.
- Tell the user when you were unsure about a platform detail and hedged it.

Don't:
- Put real eVar or event numbers, client names, URLs or IDs anywhere.
- Write full custom code where a data element and rule would do.
- Mark pages Approved, or remove the Draft status.
- Add dependencies, plugins or build steps without being asked.
- Rename files or headings that other pages link to without updating every link.
- Edit `site/` (it is generated).

## Known gaps and ideas for later

- A per-client **variable map** template (placeholder to real eVar/event numbers) is planned but not written.
- Possible additions: e-commerce, user login state, consent handling.
- Video Tracking has open questions (Adobe Media Analytics vs custom events; which players clients use). Its event is `video_milestone` on every platform, GA4 included. Do not rename it to GA4's built-in `video_progress`.
- All code and setup steps have been written from general knowledge and not yet verified on live Launch, GTM or mobile properties. Pages stay Draft until a lead tests them.
- Source material came from an example Google Analytics 4 data layer technical specification written for a client. The playbook's key names (`formId`, `universalClickthroughLabel`, `fileLinkURL`, `contentName` and so on) and firing rules were taken from it, and generalised. That spec's example in the content section uses a comma without a space for multi-value keys, which contradicts its own written rule (comma and a space). The playbook follows the written rule.
