# Error Tracking

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
Errors the visitor sees: 404 pages, server errors, failed payments. Shows where the experience breaks.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | 2 | `EVAR_ERROR_TYPE`, `EVAR_ERROR_CODE` |
    | Props | 0 | |
    | Events | 1 | `EVENT_ERROR` |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 2 | Error Type, Error Code |
    | Events | 1 | `site_error` |

Form validation errors use `form_error` from [Form Tracking](form-tracking.md). This page is for everything else.

## When to fire
When the error is shown to the visitor.

## Developer specification

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'site_error',
      'errorType': '<error type>',
      'errorCode': '<error code>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'site_error',
      'errorType': '<error type>',
      'errorCode': '<error code>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(action: "site_error", data: ["errorType": "server", "errorCode": "503"])
    ```

| Key | What it holds | Examples |
|---|---|---|
| `errorType` | Kind of error. Use the agreed list | `page not found`, `server`, `payment`, `timeout` |
| `errorCode` | A short code. Never the message text, which may contain personal data | `404`, `503` |

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element (*Data Layer Computed State*) | Path |
    |---|---|
    | `DL - errorType`, `DL - errorCode` | `errorType`, `errorCode` |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `ACDL - Error: Site Error by "site_error"` | Data layer event `site_error` | Set Variables: `EVAR_ERROR_TYPE`, `EVAR_ERROR_CODE`, event `EVENT_ERROR`. Send Beacon, custom link, `Site Error` |

=== "Adobe Web SDK (XDM)"

    XDM object with a custom `error` field group (`type`, `code`) and a custom event type such as `error.shown`. Map to `EVAR_ERROR_TYPE`, `EVAR_ERROR_CODE` and `EVENT_ERROR` in the Analytics mapping.

=== "Adobe Mobile SDK"

    Processing Rules: action `site_error` sets `EVENT_ERROR`. Context data `errorType` and `errorCode` set the two eVars.

=== "Google Analytics 4 (GTM)"

    | Item | Setup |
    |---|---|
    | Variables | `DLV - errorType`, `DLV - errorCode` |
    | Trigger | `CE - site_error` |
    | Tag | `GA4 - Site Error`, event `site_error`, parameters `error_type`, `error_code` |
    | GA4 Admin | Register both as custom dimensions |

## QA checklist
- [ ] A real 404 sends the push once
- [ ] The values are codes, not message text

## Notes
A 404 page should also send a page view with a page name that makes it easy to find (for example `...:error:404`), see [Page Naming](page-naming.md).
