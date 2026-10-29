# Site Sections

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
Groups pages into areas such as *Personal*, *Business* and *Support*, so the client can see traffic and conversions by area without reading a list of pages.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | Channel variable | 1 | Level 1 section (`s.channel`) |
    | eVars | 1 | `EVAR_SITE_SECTION`: level 1, so conversions can be split by section |
    | Props | 3 | `PROP_SECTION_L2`, `PROP_SECTION_L3`, `PROP_SECTION_L4`: sub-sections, for traffic and pathing |
    | Events | 0 | |
    | Other | | Hierarchy variable (`hier1`) is optional |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 3 | Section L2, L3, L4. Level 1 uses GA4's built-in `content_group` |
    | Events | 0 | |

## When to fire
On every page view, together with the page name.

## Values and naming
Use the same words as the page name, in the same order.

| Page name | Level 1 | L2 | L3 | L4 |
|---|---|---|---|---|
| `acme:sg:personal:cards:credit-cards` | `personal` | `cards` | `credit-cards` | |
| `acme:sg:business:loans:overview` | `business` | `loans` | `overview` | |

Leave unused levels **empty**. Do not write `n/a` or `none`.

## Developer specification
**Default: nothing for developers.** The tagging team splits the page name (see [Page Naming](page-naming.md)) into the levels. Use the keys below only if the sections cannot be worked out from the name.

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'page_view',
      'pageName': '<page name>',
      'siteSection1': '<section>',
      'siteSection2': '<sub section>',
      'siteSection3': '<sub sub section>',
      'siteSection4': '<level 4>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'page_view_custom',
      'pageName': '<page name>',
      'siteSection1': '<section>',
      'siteSection2': '<sub section>',
      'siteSection3': '<sub sub section>',
      'siteSection4': '<level 4>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(state: "acme:sg:app:cards:credit-cards", data: [
        "siteSection1": "app", "siteSection2": "cards", "siteSection3": "credit-cards"
    ])
    ```

| Key | What it holds | Example |
|---|---|---|
| `siteSection1` to `siteSection4` | Levels 1 to 4. Empty text for unused levels | `personal`, `cards`, `credit-cards`, `` |

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element | Type | Purpose |
    |---|---|---|
    | `Code - site section 1` to `Code - site section 4` | Custom code | Each returns one level from the data layer, or from the page name (below) |

    The page-load rule `PLR - Global: Page View` gets extra fields in its Set Variables action:

    | Field | Value |
    |---|---|
    | Channel | `%Code - site section 1%` |
    | `EVAR_SITE_SECTION` | `%Code - site section 1%` |
    | `PROP_SECTION_L2` to `PROP_SECTION_L4` | `%Code - site section 2%` to `%Code - site section 4%` |

    Custom code (only if needed), one data element per level (change the index, 2 for level 1):

    ```javascript
    var dl = _satellite.getVar('DL - siteSection1');
    if (dl) return dl;
    return (_satellite.getVar('Code - page name').split(':')[2] || '');  // 2 = level 1, 3 = level 2, 4 = level 3, 5 = level 4
    ```

    This assumes the page name starts with `site:country:`.

=== "Adobe Web SDK (XDM)"

    Same data elements. Level 1 goes to `web.webPageDetails.siteSection`. The other levels go to custom fields in your schema, mapped to the props in the Analytics mapping.

=== "Adobe Mobile SDK"

    Processing Rules: context data `siteSection1` sets the Channel variable and `EVAR_SITE_SECTION`. `siteSection2` to `siteSection4` set the props.

=== "Google Analytics 4 (GTM)"

    | Item | Setup |
    |---|---|
    | Variables | `DLV - siteSection1` to `DLV - siteSection4`, or custom JavaScript variables that split the page name |
    | Google tag settings | `content_group` = level 1. Parameters `section_l2`, `section_l3`, `section_l4` |
    | GA4 Admin | Register `section_l2` to `section_l4` as custom dimensions |

## QA checklist
- [ ] Levels match the page name on every template
- [ ] Empty levels send nothing
- [ ] A new area of the site does not fall into a blank section

## Notes
Props do not persist between hits, so they are for traffic and pathing. Use the eVar when you want to link a section to a later conversion.
