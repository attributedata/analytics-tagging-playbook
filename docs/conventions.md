# Placeholders and conventions

## How the playbook is organised
Our usual way of working is: **we write a specification, developers push a small JSON object to the data layer, and we build the tags.** So each tag has two parts:

1. **Developer specification.** The exact data layer push, when it fires, and what each key holds. Give this section to the website or app developers.
2. **Tagging setup.** The data elements, rules and variable mappings we create in Adobe Launch or Google Tag Manager. We only write custom code when a rule cannot do the job without it, and those snippets are labelled *Custom code (only if needed)*.

## Data layer names
| Platform | Array developers push to |
|---|---|
| Adobe Analytics and Adobe Web SDK (via Launch) | `window.adobeDataLayer` |
| Google Analytics 4 (via Google Tag Manager) | `window.dataLayer` |
| Mobile apps | No data layer. Developers call the Mobile SDK with the same keys |

The keys and event names are **identical** on every platform, so one spec serves all of them. Only the array name changes.

Always start a push with the line that creates the array, in case the library has not loaded yet:

```javascript
window.adobeDataLayer = window.adobeDataLayer || [];
```

## Data layer key rules
- Event name in `event`, lower case with underscores: `form_start`.
- Keys in camelCase: `formName`, `universalClickthroughURL`.
- Keys are **flat** (no nested objects). The data layer remembers the last value of each key, so every push must send **all** the keys for its event. Otherwise a value from an earlier event can leak into the next one.
- Values are readable text, exactly as listed in the spec (`Contact us`, not `contact_us`). We lower-case and trim them in Launch (data element options *Force lowercase value* and *Remove extra whitespace*).
- Several values in one key: separate with a comma and a space (`Supply Chain, Advanced Manufacturing`).
- No personal data. Ever.

## Placeholders instead of numbers
Clients have different variable layouts, so the playbook never says "use eVar11". It uses names in capitals:

| Placeholder | Means |
|---|---|
| `EVAR_FORM_NAME` | The eVar the client has chosen for the form name |
| `PROP_SEARCH_TERM` | The prop for the search term |
| `EVENT_FORM_START` | The success event for a form start |

When you implement for a client, fill in their real numbers in a **variable map** kept with the project:

```text
EVAR_FORM_NAME      -> eVar11
EVENT_FORM_START    -> event13
```

## The four implementation types
Every tag's tagging setup has these tabs, in this order. The tab you pick stays selected as you move between pages.

| Tab | Used when |
|---|---|
| Adobe Analytics (AppMeasurement) | Websites with the Adobe Analytics extension in Launch |
| Adobe Web SDK (XDM) | Websites or apps sending data to Adobe Experience Platform Edge |
| Adobe Mobile SDK | Native iOS and Android apps |
| Google Analytics 4 (GTM) | Websites using Google Analytics 4 through Google Tag Manager |

## Naming
- **Launch rules:** start with how it is triggered. `PLR` page load, `EBR` element-based (click, viewport, key press), `DCR` direct call, `ACDL` data layer event. Example: `ACDL - Form: Start by "form_start"`.
- **Launch data elements:** `DL - keyName` for data layer values (`DL - formName`), `Query - name` for URL parameters, `Code - name` for custom code.
- **GTM variables, triggers and tags:** `DLV - keyName` (data layer variable), `CE - event_name` (custom event trigger), `GA4 - Event Name` (tag).
- **Variable names in the analytics tool:** plain words for what the variable holds. Example: `Form Name`, not `form_nm_v2`.

## Respect privacy
Never send names, ID numbers, email addresses, phone numbers, account numbers or text typed by the visitor. If you are not sure, leave it out and ask a lead.

!!! warning "Check names against your own setup"
    Menu and field names in Launch, Adobe Experience Platform, Google Tag Manager and the Mobile SDK change between versions, and XDM schemas are often customised. Treat the setup steps as a guide and confirm them in your own property before shipping.
