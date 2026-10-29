# Site Search

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
What visitors look for, how many get results, and where search fails. Useful for content gaps, navigation fixes and measuring search-driven conversions.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | 3 | `EVAR_SEARCH_TERM`, `EVAR_SEARCH_RESULTS` (number of results as a range), `EVAR_SEARCH_TYPE` (where the search started) |
    | Props | 1 | `PROP_SEARCH_TERM`, for pathing |
    | Events | 2 | `EVENT_SEARCH` (every search), `EVENT_SEARCH_NULL` (zero results) |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 2 | Search Results, Search Type. The term uses GA4's built-in `search_term` |
    | Events | 2 | `view_search_results`, `search_no_results` |

## When to fire
Once, when the **results are shown**. Not when the visitor types or presses the button, because the result count is not known yet. If zero results are shown, push the no-results event as well.

## Developer specification

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'search_results',
      'searchTerm': '<search term>',
      'searchResultsCount': '<number of results>',
      'searchType': '<where the search started>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'view_search_results',
      'searchTerm': '<search term>',
      'searchResultsCount': '<number of results>',
      'searchType': '<where the search started>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(action: "search_results", data: [
        "searchTerm": "home loan", "searchResultsCount": "12", "searchType": "app search"
    ])
    ```

| Key | What it holds | Examples |
|---|---|---|
| `searchTerm` | What the visitor typed, trimmed | `home loan` |
| `searchResultsCount` | Number of results shown, as a number | `0`, `12` |
| `searchType` | Where the search started. Use the agreed list | `header`, `results page`, `suggestion` |

!!! info "Event name differs by platform"
    The Adobe push uses `search_results`. The Google push uses GA4's recommended name `view_search_results`. Everything else is the same.

!!! danger "Personal data in search boxes"
    People type names and account numbers into search boxes. Ask developers to remove anything that looks like an email address or a long number **before** the push. The tagging team removes them again in Launch as a safety net.

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element | Type | Path / purpose |
    |---|---|---|
    | `DL - searchTerm` | Data Layer Computed State | `searchTerm` (lower case, trimmed) |
    | `DL - searchResultsCount` | Data Layer Computed State | `searchResultsCount` |
    | `DL - searchType` | Data Layer Computed State | `searchType` |
    | `Code - search results bucket` | Custom code | Turns the count into `0`, `1-5`, `6-20`, `21+` (below) |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `ACDL - Search: Results by "search_results"` | Data layer event `search_results` | Set Variables: `EVAR_SEARCH_TERM`, `PROP_SEARCH_TERM`, `EVAR_SEARCH_RESULTS` = `%Code - search results bucket%`, `EVAR_SEARCH_TYPE`, event `EVENT_SEARCH`. Send Beacon, custom link, `Internal Search` |
    | `ACDL - Search: No results by "search_results"` | Same trigger. Condition: `DL - searchResultsCount` equals `0` | Set Variables: event `EVENT_SEARCH_NULL`. Send Beacon, `Internal Search No Results` |

    Custom code (only if needed), for the `Code - search results bucket` data element:

    ```javascript
    var n = parseInt(_satellite.getVar('DL - searchResultsCount'), 10) || 0;
    return n === 0 ? '0' : n <= 5 ? '1-5' : n <= 20 ? '6-20' : '21+';
    ```

    To remove an email address or long number, add this to the term data element's *Custom code* option, or use a Code data element: `term.replace(/\S+@\S+/g, '').replace(/\d{6,}/g, '')`.

=== "Adobe Web SDK (XDM)"

    Rule and data elements as in the AppMeasurement tab. XDM object: `eventType` `web.webinteraction.linkClicks` (or a custom `search.results`), search fields in your custom `search` field group: `term`, `resultsCount`, `type`. Map them to the eVars and prop in the Analytics mapping. Use the same results bucket data element.

=== "Adobe Mobile SDK"

    Processing Rules:

    | Condition | Set |
    |---|---|
    | Action name equals `search_results` | `EVENT_SEARCH` |
    | Action name equals `search_results` and context data `searchResultsCount` equals `0` | `EVENT_SEARCH_NULL` |
    | Context data `searchTerm` exists | `EVAR_SEARCH_TERM` and `PROP_SEARCH_TERM` |
    | Context data `searchType` exists | `EVAR_SEARCH_TYPE` |

    Ask the developers to send the results bucket (`0`, `1-5`, `6-20`, `21+`) as `searchResultsCount`, or add a processing rule that sets `EVAR_SEARCH_RESULTS` from the number.

=== "Google Analytics 4 (GTM)"

    | Item | Setup |
    |---|---|
    | Variables | `DLV - searchTerm`, `DLV - searchResultsCount`, `DLV - searchType` |
    | Trigger | `CE - view_search_results` |
    | Tag | `GA4 - View Search Results`, event `view_search_results`, parameters `search_term`, `search_results`, `search_type` |
    | Second trigger and tag | Same trigger plus condition `DLV - searchResultsCount` equals `0`; tag `GA4 - Search No Results`, event `search_no_results` |
    | GA4 Admin | Register `search_results` and `search_type` as custom dimensions |

    If the site's search uses a URL parameter such as `?q=`, GA4's enhanced measurement can collect `view_search_results` automatically. Turn that off when you use the data layer push, to avoid duplicates.

## QA checklist
- [ ] A search with results sends one search hit and no null event
- [ ] A search with no results also sends the null event
- [ ] The term is lower case and trimmed
- [ ] Typing an email address into search does not reach any tool
- [ ] Refreshing the results page does not count a second search

## Notes
Use `searchType` `suggestion` for type-ahead suggestions so you can compare them with full searches.
