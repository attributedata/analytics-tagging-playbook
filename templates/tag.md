# Topic name

Status: **Draft** · Owner: **Your name** · Last reviewed: **DD-Mon-YYYY**
{ .page-meta }

## What this tracks and why
Two or three sentences. What question does this answer for the business?

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | 0 | `EVAR_EXAMPLE`: description |
    | Props | 0 | |
    | Events | 0 | `EVENT_EXAMPLE`: when it fires |
    | Other | | |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 0 | |
    | Events | 0 | |

## When to fire
Describe the trigger in plain words.

## Developer specification
Give this section to the developers.

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'event_name',
      'keyOne': '<value>',
      'keyTwo': '<value>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'event_name',
      'keyOne': '<value>',
      'keyTwo': '<value>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(action: "event_name", data: ["keyOne": "<value>", "keyTwo": "<value>"])
    ```

| Key | What it holds | Expected values and examples |
|---|---|---|
| `keyOne` | | |

**Example**

```json
{ "event": "event_name", "keyOne": "example value" }
```

## Tagging setup
Data elements and rules. Add code only if a rule cannot do the job.

=== "Adobe Analytics (AppMeasurement)"

    | Data element | Type | Path |
    |---|---|---|
    | `DL - keyOne` | Data Layer Computed State | `keyOne` |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `ACDL - Topic: Name by "event_name"` | Data layer event `event_name` | Set Variables: ... then Send Beacon |

=== "Adobe Web SDK (XDM)"

    Rule, XDM object and datastream mapping.

=== "Adobe Mobile SDK"

    Processing rules.

=== "Google Analytics 4 (GTM)"

    Variables, trigger, tag, custom dimensions.

## QA checklist
- [ ] What you should see in the debugger
- [ ] Edge cases tested

## Notes
Anything that surprised you.
