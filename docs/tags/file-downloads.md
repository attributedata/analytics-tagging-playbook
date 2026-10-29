# File Downloads

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
Which guides, reports and brochures people download.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | 3 | `EVAR_FILE_NAME`, `EVAR_FILE_TITLE`, `EVAR_FILE_URL` |
    | Props | 0 | |
    | Events | 1 | `EVENT_FILE_DOWNLOAD` |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 3 | File Name, File Title, File URL |
    | Events | 1 | `file_download` |

The link text goes into the custom link name of the beacon, so it needs no variable.

## When to fire
When the visitor clicks a link that downloads a resource (PDF guides, setup guides, profiles, spreadsheets).

## Developer specification

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'file_download',
      'fileTitle': '<file title>',
      'fileName': '<file name>',
      'fileLinkText': '<file link text>',
      'fileLinkURL': '<file link URL>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'file_download',
      'fileTitle': '<file title>',
      'fileName': '<file name>',
      'fileLinkText': '<file link text>',
      'fileLinkURL': '<file link URL>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(action: "file_download", data: [
        "fileTitle": "Annual Sustainability Report",
        "fileName": "annual-sustainability-report-2026",
        "fileLinkText": "Download (8.6 MB)",
        "fileLinkURL": "https://assets.example.com/reports/annual-sustainability-report-2026.pdf"
    ])
    ```

| Key | What it holds | Example |
|---|---|---|
| `fileTitle` | Short description of the file | `Annual Sustainability Report` |
| `fileName` | File name from the URL, without the extension | URL `.../reports/annual-sustainability-report-2026.pdf` gives `annual-sustainability-report-2026` |
| `fileLinkText` | The text of the link clicked | `Download (8.6 MB)` |
| `fileLinkURL` | The full `href` of the link | `https://assets.example.com/reports/annual-sustainability-report-2026.pdf` |

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element (*Data Layer Computed State*) | Path |
    |---|---|
    | `DL - fileTitle`, `DL - fileName`, `DL - fileLinkText`, `DL - fileLinkURL` | `fileTitle`, `fileName`, `fileLinkText`, `fileLinkURL` |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `ACDL - Link: File Download by "file_download"` | Data layer event `file_download` | Set Variables: `EVAR_FILE_NAME`, `EVAR_FILE_TITLE`, `EVAR_FILE_URL`, event `EVENT_FILE_DOWNLOAD`. Send Beacon: link type **Download Link**, link name `%DL - fileLinkText%` |

=== "Adobe Web SDK (XDM)"

    XDM object: `eventType` `web.webinteraction.linkClicks`, `web.webInteraction.type` `download`, `name` = `DL - fileTitle`, `URL` = `DL - fileLinkURL`, `linkClicks.value` 1. Add a custom field for the file name. Map the three fields to the eVars in the Analytics mapping.

=== "Adobe Mobile SDK"

    Processing Rules: action name `file_download` sets `EVENT_FILE_DOWNLOAD`. Context data `fileName`, `fileTitle`, `fileLinkURL` set `EVAR_FILE_NAME`, `EVAR_FILE_TITLE`, `EVAR_FILE_URL`.

=== "Google Analytics 4 (GTM)"

    | Item | Setup |
    |---|---|
    | Variables | `DLV - fileTitle`, `DLV - fileName`, `DLV - fileLinkText`, `DLV - fileLinkURL` |
    | Trigger | `CE - file_download` |
    | Tag | `GA4 - File Download`, event `file_download`, parameters `file_title`, `file_name`, `link_text`, `link_url` |
    | GA4 Admin | Register `file_title`, `file_name`, `link_url` as custom dimensions |

    !!! warning "Turn off enhanced measurement file downloads"
        GA4 collects `file_download` automatically (with its own `file_name` and `link_url` parameters). Turn off *File downloads* in enhanced measurement, or each download is counted twice.

## QA checklist
- [ ] Each file type sends one hit
- [ ] File name has no extension and no query string
- [ ] Link text and URL match what the visitor clicked
- [ ] Right-click "Save as" is not counted. Tell clients this is a known gap

## Notes
A download link that also triggers `universal_click` must send only `file_download`.
