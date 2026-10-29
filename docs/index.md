# Analytics Tagging Playbook

Our standard way of tagging websites and apps with Adobe Analytics and Google Analytics 4. Use it before you build a new implementation, when you review someone else's, and when a client asks "how should we track this?".

Every tag has two parts:

- A **developer specification**: the data layer push, when to fire it, and what each key holds. Give this to the client's developers.
- A **tagging setup** for our team: the data elements, rules and variable mappings for each implementation type. Custom code only where needed.

| Implementation | Developers push to |
|---|---|
| Adobe Analytics (AppMeasurement) | `adobeDataLayer` |
| Adobe Web SDK (XDM) | `adobeDataLayer` |
| Adobe Mobile SDK | Mobile SDK calls, same keys |
| Google Analytics 4 (GTM) | `dataLayer` |

## Where to start

| I need to track... | Read | eVars | Events | GA4 dimensions |
|---|---|---|---|---|
| Installing the tag manager | [Library Installation](tags/embed-code.md) | | | |
| Where visitors came from | [Marketing Channels](tags/marketing-channels.md) | 0 | 0 | 0 |
| What page they are on | [Page Naming](tags/page-naming.md) | 1 | 0 | 1 |
| Which part of the site | [Site Sections](tags/site-sections.md) | 1 | 0 | 3 |
| What they searched for | [Site Search](tags/site-search.md) | 3 | 2 | 2 |
| Forms started and completed | [Form Tracking](tags/form-tracking.md) | 4 | 4 | 4 |
| Any link, button or banner click | [Universal Clickthrough](tags/clickthrough.md) | 3 | 1 | 3 |
| Downloads | [File Downloads](tags/file-downloads.md) | 3 | 1 | 3 |
| Errors | [Error Tracking](tags/error-tracking.md) | 2 | 1 | 2 |
| Video plays and how much was watched | [Video Tracking](tags/video-tracking.md) | 2 | 5 | 2 |
| Article and content metadata | [Content Page Tracking](tags/content-pages.md) | up to 9 | 1 | up to 9 |

!!! note "These pages are drafts"
    Every page is marked **Draft** until a team lead reviews it. The counts are our recommended starting point, not Adobe or Google rules. Check them against each client's available variables.

## Rules of the playbook

1. Tags never use real eVar or event numbers. They use [placeholders](conventions.md) like `EVAR_FORM_NAME`, so they work for any client.
2. Never send personal data (names, ID numbers, account numbers, email addresses) in any variable.
3. Keys and event names are the same on every platform. Only the data layer array changes.
4. If a page is wrong or missing something, fix it. See [How to contribute](contributing.md).
