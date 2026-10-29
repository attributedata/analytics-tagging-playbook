# Page Naming

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
Every page view needs a clear, stable page name. It is the main label in Pathing, Fallout and Page reports. A bad or changing name makes years of data hard to compare.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | Page Name | 1 | Page name (`s.pageName`) |
    | eVars | 1 | `EVAR_PAGE_NAME`: the same name, so it can be used with conversion events |
    | Props | 0 (1 optional) | `PROP_PAGE_NAME`: only if you need pathing on a prop |
    | Events | 0 | |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 1 | Page Name |
    | Events | 0 | Uses the built-in `page_view` |

## When to fire
On every page view and every virtual page view (single-page apps and pages that change content without reloading).

## Values and naming
Format: `site:section:subsection:page`, lower case, colon between levels, hyphens inside words, under 100 characters.

| Good | Bad |
|---|---|
| `acme:sg:personal:cards:credit-cards` | `Credit Cards` |
| `acme:sg:business:loans:overview` | `https://acme.com/business/loans/index.html` |

Never use the full URL, the page title (marketing edits titles), query strings, IDs or personal data.

## Developer specification
**Default: nothing for developers.** The tagging team builds the name from the URL path in one data element (see Tagging setup). Use the push below only when the name cannot be worked out from the URL, for example for single-page apps or pages with generic URLs. The same push can also carry the [site section](site-sections.md) keys.

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'page_view',
      'pageName': '<page name>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'page_view_custom',
      'pageName': '<page name>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(state: "acme:sg:app:cards:credit-cards", data: nil)
    ```

    ```kotlin
    MobileCore.trackState("acme:sg:app:cards:credit-cards", null)
    ```

| Key | What it holds | Example |
|---|---|---|
| `pageName` | The page name in the format above | `acme:sg:personal:cards:credit-cards` |

Run the push once the page content has loaded, and again for each virtual page view.

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element | Type | Purpose |
    |---|---|---|
    | `DL - pageName` | Data Layer Computed State, path `pageName` | Name from the data layer, if developers push it |
    | `Code - page name` | Custom code | Name from the URL, the default (below) |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `PLR - Global: Page View` | Page Bottom (or `page_view` data layer event for single-page apps) | Set Variables: Page Name = `%Code - page name%`, `EVAR_PAGE_NAME` = `%Code - page name%`. Send Beacon, page view |

    Custom code (only if needed), for `Code - page name`. It prefers the data layer and falls back to the URL:

    ```javascript
    var dl = _satellite.getVar('DL - pageName');
    if (dl) return dl;
    var path = location.pathname.replace(/\.(html|page|aspx)$/, '')
                                .replace(/\/index$/, '')
                                .replace(/^\/|\/$/g, '');
    var parts = path ? path.split('/') : ['home'];
    return ['acme'].concat(parts).join(':').toLowerCase();
    ```

=== "Adobe Web SDK (XDM)"

    Same data elements. In the XDM object for the page view, set `eventType` to `web.webpagedetails.pageViews`, `web.webPageDetails.name` to `%Code - page name%` and `web.webPageDetails.pageViews.value` to `1`. In Analytics, `web.webPageDetails.name` becomes the Page Name. Copy the name to `EVAR_PAGE_NAME` through the mapping.

=== "Adobe Mobile SDK"

    Developers use screen names in the same format. No processing rule is needed for the Page Name. If a copy in `EVAR_PAGE_NAME` is needed, add a rule: *Page Name exists, set `EVAR_PAGE_NAME` = Page Name*.

=== "Google Analytics 4 (GTM)"

    GA4 sends `page_view` with the page title and URL automatically. Add the name as an extra parameter:

    | Item | Setup |
    |---|---|
    | Variable | `DLV - pageName`, or a *Custom JavaScript* variable that builds the name from the URL |
    | Google tag settings | Add a parameter `page_name` = that variable |
    | GA4 Admin | Register `page_name` as an event-scoped custom dimension |

## QA checklist
- [ ] Name matches the format on 5 page types (home, listing, article, form, error)
- [ ] Only one page view per page
- [ ] The name does not change after a language switch, unless intended
- [ ] Virtual page views send a new name

## Notes
Changing page names later splits your reports. Agree the format with the client **before** launch.
