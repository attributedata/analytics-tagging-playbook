# Universal Clickthrough

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
One consistent way to track clicks on links, buttons and banners that lead to a new page, without building a rule for every element. It answers "what do people click, and where does it take them?".

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | 3 | `EVAR_CLICK_COMPONENT`, `EVAR_CLICK_LABEL`, `EVAR_CLICK_URL` |
    | Props | 0 | |
    | Events | 1 | `EVENT_UNIVERSAL_CLICK` |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 3 | Click Component, Click Label, Click URL |
    | Events | 1 | `universal_click` |

## When to fire
On the click of **any link on the site that leads to a new page**. Downloads use [File Downloads](file-downloads.md) instead.

## Developer specification

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'universal_click',
      'universalClickthroughComponent': '<component type>',
      'universalClickthroughLabel': '<label>',
      'universalClickthroughURL': '<destination URL>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'universal_click',
      'universalClickthroughComponent': '<component type>',
      'universalClickthroughLabel': '<label>',
      'universalClickthroughURL': '<destination URL>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(action: "universal_click", data: [
        "universalClickthroughComponent": "button",
        "universalClickthroughLabel": "Get in touch",
        "universalClickthroughURL": "https://www.example.com/contact-us"
    ])
    ```

| Key | What it holds | Examples |
|---|---|---|
| `universalClickthroughComponent` | The type of UI element clicked | `link`, `button`, `banner` |
| `universalClickthroughLabel` | The text shown on the element | `Get in touch` |
| `universalClickthroughURL` | The full `href` of the link | `https://www.example.com/contact-us` |

**Example**

```json
{
  "event": "universal_click",
  "universalClickthroughComponent": "button",
  "universalClickthroughLabel": "Get in touch",
  "universalClickthroughURL": "https://www.example.com/contact-us"
}
```

!!! tip "If developers cannot add the push"
    A Launch **click** rule on `a[href]` can read the element's text and `href` itself, with no developer work. It is less reliable (text changes, buttons that are not links) and the component type has to be guessed, so use it only as a stop-gap.

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element (*Data Layer Computed State*) | Path |
    |---|---|
    | `DL - universalClickthroughComponent` | `universalClickthroughComponent` |
    | `DL - universalClickthroughLabel` | `universalClickthroughLabel` |
    | `DL - universalClickthroughURL` | `universalClickthroughURL` |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `ACDL - Link: Universal Click by "universal_click"` | Data layer event `universal_click` | Set Variables: `EVAR_CLICK_COMPONENT`, `EVAR_CLICK_LABEL`, `EVAR_CLICK_URL`, event `EVENT_UNIVERSAL_CLICK`. Then Send Beacon, custom link, name `Universal Click` |

    Keep the labels lower case (data element option) so `Get in touch` and `Get in Touch` become one value.

=== "Adobe Web SDK (XDM)"

    Same trigger and data elements. In the XDM object use the link fields that already exist:

    | XDM field | Source |
    |---|---|
    | `eventType` | `web.webinteraction.linkClicks` |
    | `web.webInteraction.name` | `DL - universalClickthroughLabel` |
    | `web.webInteraction.URL` | `DL - universalClickthroughURL` |
    | `web.webInteraction.type` | `other` |
    | `web.webInteraction.linkClicks.value` | `1` |
    | `_tenant.click.component` (custom) | `DL - universalClickthroughComponent` |

    Map `web.webInteraction.name` and `URL` to `EVAR_CLICK_LABEL` and `EVAR_CLICK_URL`, and the custom field to `EVAR_CLICK_COMPONENT`, in your Analytics mapping.

=== "Adobe Mobile SDK"

    Processing Rules:

    | Condition | Set |
    |---|---|
    | Action name equals `universal_click` | `EVENT_UNIVERSAL_CLICK` |
    | Context data `universalClickthroughComponent` exists | `EVAR_CLICK_COMPONENT` |
    | Context data `universalClickthroughLabel` exists | `EVAR_CLICK_LABEL` |
    | Context data `universalClickthroughURL` exists | `EVAR_CLICK_URL` |

=== "Google Analytics 4 (GTM)"

    | Item | Setup |
    |---|---|
    | Variables | `DLV - universalClickthroughComponent`, `DLV - universalClickthroughLabel`, `DLV - universalClickthroughURL` |
    | Trigger | `CE - universal_click`, Custom Event, event name `universal_click` |
    | Tag | `GA4 - Universal Click`, GA4 Event `universal_click`, parameters `click_component`, `click_label`, `click_url` |
    | GA4 Admin | Register the three parameters as event-scoped custom dimensions |

    GA4 collects its own `click` event for outbound links when enhanced measurement is on. Keep it on only if you want both.

## QA checklist
- [ ] Click a link: one hit, with the right component, label and URL
- [ ] Click an icon inside a link: still one hit
- [ ] The URL is the full destination, not a relative path
- [ ] Download links send `file_download`, not `universal_click`

## Notes
Clicks that open in a new tab must send the hit before the page changes. Test with the Network tab set to *Preserve log*.
