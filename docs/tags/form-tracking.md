# Form Tracking

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
How many people start a form, how far they get, how many finish, and where errors happen. Forms are usually the main conversion on a site.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | 4 | `EVAR_FORM_ID`, `EVAR_FORM_TYPE`, `EVAR_FORM_NAME`, `EVAR_FORM_STEP` |
    | Props | 0 | |
    | Events | 4 | `EVENT_FORM_START`, `EVENT_FORM_STEP`, `EVENT_FORM_COMPLETE`, `EVENT_FORM_ERROR` |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 4 | Form ID, Form Type, Form Name, Form Step |
    | Events | 4 | `form_start`, `form_step`, `form_complete`, `form_error` |

Single-step forms need only start and complete. Skip the step and error events and their variables if the client does not need them.

## When to fire
| Event | Fires when |
|---|---|
| `form_start` | The visitor's **first ever interaction** with any field in the form. Once per form per visit |
| `form_step` | A step is completed and the next screen is shown (multi-step forms) |
| `form_complete` | **After the success response (HTTP 200) is received**, meaning the submission was accepted. Not when the button is clicked |
| `form_error` | A validation or submission error is shown to the visitor |

!!! note "Forms with a topic or type selector"
    If the form name depends on a dropdown (for example "Select a topic") and the visitor changes it, fire `form_start` again with the new `formName`.

## Developer specification

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'form_start',
      'formId': '<CMS form ID>',
      'formType': '<form type>',
      'formName': '<form name>'
    });
    ```

    On `form_complete`, add the submission ID:

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'form_complete',
      'formId': '<CMS form ID>',
      'formType': '<form type>',
      'formName': '<form name>',
      'submissionId': '<submission ID>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'form_start',
      'formId': '<CMS form ID>',
      'formType': '<form type>',
      'formName': '<form name>'
    });
    ```

    On `form_complete`, add the submission ID:

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'form_complete',
      'formId': '<CMS form ID>',
      'formType': '<form type>',
      'formName': '<form name>',
      'submissionId': '<submission ID>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(action: "form_start", data: [
        "formId": "<CMS form ID>", "formType": "<form type>", "formName": "<form name>"
    ])
    ```

    ```kotlin
    MobileCore.trackAction("form_complete", mapOf(
        "formId" to "<CMS form ID>", "formType" to "<form type>",
        "formName" to "<form name>", "submissionId" to "<submission ID>"
    ))
    ```

| Key | Sent on | What it holds | Examples |
|---|---|---|---|
| `formId` | all four events | ID of the form in the CMS or form tool | `contact-us-01` |
| `formType` | all | The kind of form. Use the agreed list | `Contact us`, `Subscription`, `Workshop registration` |
| `formName` | all | The name of the form, or the selected variant | `General enquiry`, `Subscription` |
| `formStep` | `form_step`, `form_error` | Step number and name | `2 - Company details` |
| `submissionId` | `form_complete` | Unique ID for this submission, for example the reference returned by the server | `SUB-20261029-0042` |

**Example (`form_complete`)**

```json
{
  "event": "form_complete",
  "formId": "contact-us-01",
  "formType": "Contact us",
  "formName": "General enquiry",
  "submissionId": "SUB-20261029-0042"
}
```

!!! danger "Never send what the visitor typed"
    Send only the keys above. Never send field values, not even an email address, a name or a hash of them. `submissionId` must be a reference number and must not contain personal data.

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    **Data elements** (Adobe Client Data Layer extension, type *Data Layer Computed State*). Turn on *Force lowercase value* and *Remove extra whitespace*.

    | Name | Path |
    |---|---|
    | `DL - formId` | `formId` |
    | `DL - formType` | `formType` |
    | `DL - formName` | `formName` |
    | `DL - formStep` | `formStep` |
    | `DL - submissionId` | `submissionId` |

    **Rules.** Trigger: Adobe Client Data Layer, *Data pushed*, event `form_start` (the label may differ slightly by extension version).

    | Rule | Trigger event | Set Variables action | Then |
    |---|---|---|---|
    | `ACDL - Form: Start by "form_start"` | `form_start` | `EVAR_FORM_ID`, `EVAR_FORM_TYPE`, `EVAR_FORM_NAME` = the data elements. Event `EVENT_FORM_START` | Send Beacon, custom link, name `Form Start` |
    | `ACDL - Form: Step by "form_step"` | `form_step` | As above plus `EVAR_FORM_STEP`. Event `EVENT_FORM_STEP` | Send Beacon, `Form Step` |
    | `ACDL - Form: Complete by "form_complete"` | `form_complete` | As start. Event `EVENT_FORM_COMPLETE` with **event serialization** set to `%DL - submissionId%` | Send Beacon, `Form Complete` |
    | `ACDL - Form: Error by "form_error"` | `form_error` | As step. Event `EVENT_FORM_ERROR` | Send Beacon, `Form Error` |

    No custom code is needed. The serialization option is in the event field of the Set Variables action. It stops a refresh of the thank-you page counting twice (see [Event Serialization](event-serialization.md)).

=== "Adobe Web SDK (XDM)"

    1. Create an **XDM object** data element per event, for example `XDM - Form Complete`, with `eventType` set to an agreed value (`form.complete`) and your form fields filled from the `DL -` data elements.
    2. Rule: same trigger as the AppMeasurement tab. Action: Adobe Experience Platform Web SDK, *Send event*, with that XDM object.
    3. Map the fields to the Analytics variables, either with Processing Rules on the context data, or by sending `data.__adobe.analytics` in the event's Data object.

    | XDM field (in your tenant namespace) | Source | Analytics variable |
    |---|---|---|
    | `form.id` | `DL - formId` | `EVAR_FORM_ID` |
    | `form.type` | `DL - formType` | `EVAR_FORM_TYPE` |
    | `form.name` | `DL - formName` | `EVAR_FORM_NAME` |
    | `form.step` | `DL - formStep` | `EVAR_FORM_STEP` |
    | `eventType` | | Event `EVENT_FORM_*` |

    Add the custom `form` field group and the event type values to your schema first.

=== "Adobe Mobile SDK"

    Developers send the calls in the spec above. In Adobe Analytics, add **Processing Rules** for the report suite (*Admin > Report Suites > Edit Settings > General > Processing Rules*):

    | Condition | Set |
    |---|---|
    | Action name equals `form_start` | `EVENT_FORM_START` |
    | Action name equals `form_step` | `EVENT_FORM_STEP` |
    | Action name equals `form_complete` | `EVENT_FORM_COMPLETE`, serial from context data `submissionId` |
    | Action name equals `form_error` | `EVENT_FORM_ERROR` |
    | Context data `formId` exists | `EVAR_FORM_ID` = `formId` |
    | Context data `formType` exists | `EVAR_FORM_TYPE` = `formType` |
    | Context data `formName` exists | `EVAR_FORM_NAME` = `formName` |
    | Context data `formStep` exists | `EVAR_FORM_STEP` = `formStep` |

    If the app sends data through the Edge Network instead, follow the Web SDK tab and send the same fields in the XDM event.

=== "Google Analytics 4 (GTM)"

    **Variables** (type *Data Layer Variable*, Data Layer Version 2): `DLV - formId`, `DLV - formType`, `DLV - formName`, `DLV - formStep`, `DLV - submissionId`.

    **Triggers** (type *Custom Event*): `CE - form_start`, `CE - form_step`, `CE - form_complete`, `CE - form_error`, each with the event name as the trigger name.

    **Tags** (type *GA4 Event*): one per trigger, event name equal to the trigger's event. Add event parameters:

    | Parameter | Value |
    |---|---|
    | `form_id` | `{{DLV - formId}}` |
    | `form_type` | `{{DLV - formType}}` |
    | `form_name` | `{{DLV - formName}}` |
    | `form_step` | `{{DLV - formStep}}` (step and error tags) |
    | `submission_id` | `{{DLV - submissionId}}` (complete tag; used for de-duplication, not as a dimension) |

    Then, in GA4 *Admin > Custom definitions*, register `form_id`, `form_type`, `form_name` and `form_step` as event-scoped **custom dimensions**, and mark `form_complete` as a **key event**.

    !!! warning "Turn off enhanced measurement forms"
        GA4 can collect `form_start` and `form_submit` by itself. Turn off *Form interactions* in enhanced measurement, or you will count starts twice.

## QA checklist
- [ ] Start fires once, not on every field
- [ ] Changing a form topic fires a new start with the new name
- [ ] Complete fires only on success, never on a failed submit
- [ ] Refreshing the thank-you page does not count a second complete
- [ ] Every push has all its keys
- [ ] No typed values appear in any request

## Notes
- Put the form name in the data layer. Do not read the `<h1>` or page title.
- One pattern for every form. Do not create a separate set of rules for each form.
