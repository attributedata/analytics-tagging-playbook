# Event Serialization

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
Stops the same conversion being counted twice, for example when a visitor refreshes a thank-you page. Each submission has a unique ID, and Adobe ignores a repeat of the same ID on the same event.

## Variables required
None. It uses the events you already have.

=== "Adobe Analytics"

    | Type | How many | Notes |
    |---|---|---|
    | eVars | 0 | |
    | Events | 0 | Turn on serialization for the event in Admin (*Success Events*, serialization: use event ID) |

=== "Google Analytics 4"

    | Type | How many | Notes |
    |---|---|---|
    | Custom dimensions | 0 | |
    | Events | 0 | GA4 has no event serialization. See the Google Analytics 4 tab under Tagging setup |

## When to fire
On events that must count once per action: form complete, application complete, purchase.

## Developer specification
The developers send a unique ID in the same push as the event. For forms this is `submissionId` (see [Form Tracking](form-tracking.md)).

| Key | What it holds | Example |
|---|---|---|
| `submissionId` | A unique reference for this action, ideally returned by the server. Same value if the thank-you page reloads. Short, no personal data | `SUB-20261029-0042` |

If the server cannot supply one, the developers can create it once and keep it for the visit:

```javascript
var id = sessionStorage.getItem('submissionId') || Date.now().toString(36);
sessionStorage.setItem('submissionId', id);
```

They must clear it when a new submission starts.

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    In the Set Variables action, in the *Events* section, select the event (for example `EVENT_FORM_COMPLETE`) and set **Event serialization** to `Use event ID`, with the value `%DL - submissionId%`. No code is needed. Keep the ID under 20 characters.

=== "Adobe Web SDK (XDM)"

    Send the event and the ID through the Analytics data in the event (`events: 'EVENT_FORM_COMPLETE:<id>'`), or use the datastream's mapping. Check how your datastream passes events. Edge has its own event ID for deduplicating requests; do not confuse the two.

=== "Adobe Mobile SDK"

    In Processing Rules, set the event with the serial value from context data `submissionId`.

=== "Google Analytics 4 (GTM)"

    GA4 has no event serialization. Prevent duplicates instead:

    - Fire the event only from the `form_complete` push, not on page load of the thank-you page.
    - In the GTM tag, add a trigger exception: do not fire if a first-party cookie or `sessionStorage` value equals `submissionId` (a small Custom JavaScript variable can check this).
    - In GA4 reports, count unique `submission_id` values with an exploration if you need exact numbers.

## QA checklist
- [ ] Submit once: the event has an ID
- [ ] Refresh the thank-you page: the report counts it once
- [ ] A second real submission gets a different ID

## Notes
Serialization works per event and per report suite. Test it on the event in the report, not only in the debugger, because duplicates are removed during processing.
