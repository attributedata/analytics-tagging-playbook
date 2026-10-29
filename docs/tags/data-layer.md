# Data Layer

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this is and why
A data layer is a JavaScript array that the website pushes small objects into. Tags listen to the pushes and read the values, instead of scraping the page. When the site's design changes, tracking keeps working.

We use `adobeDataLayer` for Adobe and `dataLayer` for Google Tag Manager.

## Variables required
None. The data layer feeds the variables on the other pages.

## How it works
```javascript
window.adobeDataLayer = window.adobeDataLayer || [];
window.adobeDataLayer.push({
  'event': 'form_start',
  'formName': 'General enquiry'
});
```

1. The first line creates the array if it does not exist yet.
2. Each `push` sends one object. The object with an `event` key tells the tag manager that something happened.
3. A rule listens for that event name and reads the other keys.

## Rules for developers
- **One push per moment.** One push for a form start, one for a click. Do not combine them.
- **Flat keys.** No nested objects.
- **Send every key every time.** The data layer remembers the last value of each key. If a push leaves a key out, the old value stays and can leak into the next event.
- **Fire at the right time.** Pushes for "success" events run after the server confirms.
- **Plain text values.** Use the readable values from the spec. No personal data.
- **Names exactly as spec'd.** Keys are camelCase (`formName`), events are lower case with underscores (`form_start`).

## Developer specification
See the individual tags. The standard events are:

| Event | Tag |
|---|---|
| `page_view` (optional) | [Page Naming](page-naming.md), [Site Sections](site-sections.md) |
| `search_results` (Adobe) / `view_search_results` (Google) | [Site Search](site-search.md) |
| `form_start`, `form_step`, `form_complete`, `form_error` | [Form Tracking](form-tracking.md) |
| `universal_click` | [Universal Clickthrough](clickthrough.md) |
| `file_download` | [File Downloads](file-downloads.md) |
| `site_error` | [Error Tracking](error-tracking.md) |
| `video_milestone` | [Video Tracking](video-tracking.md) |
| `content_view` (optional) | [Content Page Tracking](content-pages.md) |

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    Add the **Adobe Client Data Layer** extension in Launch. Create data elements of type *Data Layer Computed State* with the key as the path. Create rules triggered by the extension's data layer event, set to the event name. Menu names can differ slightly by extension version.

=== "Adobe Web SDK (XDM)"

    Same as the AppMeasurement tab for triggers and data elements. The XDM object data element then gathers the values into the XDM structure.

=== "Adobe Mobile SDK"

    There is no data layer in apps. Developers call `MobileCore.track(action:data:)` (iOS) or `MobileCore.trackAction` (Android) with the same keys.

=== "Google Analytics 4 (GTM)"

    Create *Data Layer Variable* variables (Data Layer Version 2) and *Custom Event* triggers named after the event.

## QA checklist
- [ ] In the browser console, type `adobeDataLayer` (or `dataLayer`): the pushes are listed
- [ ] In Adobe, `adobeDataLayer.getState()` returns the combined values
- [ ] The data layer exists before the library loads
- [ ] Values change correctly on virtual page views

## Notes
Adobe also offers an event-driven data layer standard (Adobe Client Data Layer). The pushes shown here are compatible with it.
