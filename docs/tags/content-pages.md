# Content Page Tracking

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
Describes articles, guides, case studies and similar content so the client can see which topics, industries and formats perform. Only for **content pages**. Normal pages (home, forms, product pages) must not fire it.

This is an optional topic. Choose only the keys the client will report on.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | Up to 9 | One per key you keep: `EVAR_CONTENT_NAME`, `EVAR_CONTENT_TYPE`, `EVAR_CONTENT_TOPIC`, `EVAR_CONTENT_SEC_TOPICS`, `EVAR_CONTENT_INDUSTRY`, `EVAR_CONTENT_SOURCE`, `EVAR_CONTENT_PUB_DATE`, `EVAR_CONTENT_LENGTH`, `EVAR_CONTENT_FILE_SIZE` |
    | Props | 0 | |
    | Events | 1 | `EVENT_CONTENT_VIEW` |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | Up to 9 | One per key kept |
    | Events | 1 | `content_view` |

A typical set is 5 to 6 keys: name, type, topic, industry, source and publication date.

## When to fire
Once the content page has loaded. Fire it **in addition to** the normal page view.

## Developer specification

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'content_view',
      'contentName': '<content name>',
      'contentType': '<content type>',
      'contentTopic': '<content topic>',
      'contentSecTopics': '<content secondary topics>',
      'contentIndustry': '<content industry>',
      'contentSource': '<content source>',
      'contentPublicationDate': '<content publication date>',
      'contentLengthByTime': '<content length by time>',
      'contentFileSize': '<content file size>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'content_view',
      'contentName': '<content name>',
      'contentType': '<content type>',
      'contentTopic': '<content topic>',
      'contentSecTopics': '<content secondary topics>',
      'contentIndustry': '<content industry>',
      'contentSource': '<content source>',
      'contentPublicationDate': '<content publication date>',
      'contentLengthByTime': '<content length by time>',
      'contentFileSize': '<content file size>'
    });
    ```

=== "Mobile apps"

    ```swift
    MobileCore.track(action: "content_view", data: [
        "contentName": "<content name>", "contentType": "<content type>", "contentTopic": "<content topic>"
    ])
    ```

| Key | What it holds | Expected values and examples |
|---|---|---|
| `contentName` | Name of the content page | The article title, or the topic or category name for listing pages |
| `contentType` | Type of content. Use the agreed list | `Insights article`, `Case study article`, `Business guide article` |
| `contentTopic` | Primary topic. One value from the agreed list | `Sustainability` |
| `contentSecTopics` | Secondary topics. Several values separated by a comma and a space | `Supply Chain, Technology` |
| `contentIndustry` | Industry focus. One value from the agreed list | `Logistics & Supply Chain Management` |
| `contentSource` | Where the content came from | `Original`, `Partnered`, `Syndicated`, `Sponsored` |
| `contentPublicationDate` | Date in `dd mmm yyyy` | `27 Apr 2026` |
| `contentLengthByTime` | Reading time | `8 min` |
| `contentFileSize` | Download size (for e-books). Empty text if none | `197KB`, `5MB` |

The lists for type, topic, industry and source belong to each client. Keep them in the client's own spec, and use the same values everywhere. Send every key, with empty text (`''`) when it does not apply.

**Example**

```json
{
  "event": "content_view",
  "contentName": "Five ways to reinvent your business",
  "contentType": "Insights article",
  "contentTopic": "Innovation",
  "contentSecTopics": "Supply Chain, Technology",
  "contentIndustry": "General",
  "contentSource": "Original",
  "contentPublicationDate": "27 Apr 2026",
  "contentLengthByTime": "8 min",
  "contentFileSize": ""
}
```

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element (*Data Layer Computed State*) | Path |
    |---|---|
    | `DL - contentName` and so on, one per key | `contentName`, `contentType`, `contentTopic`, `contentSecTopics`, ... |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `ACDL - Content: View by "content_view"` | Data layer event `content_view` | Set Variables: each `EVAR_CONTENT_*` = its data element, event `EVENT_CONTENT_VIEW`. Send Beacon, custom link, `Content View` |

    Multi-value keys stay as one text value. If the client wants to report each secondary topic separately, use a list eVar (delimiter `,`) in Adobe Admin.

=== "Adobe Web SDK (XDM)"

    XDM object with a custom `content` field group, one field per key, and event type `content.view`. Map the fields to the eVars in the Analytics mapping.

=== "Adobe Mobile SDK"

    Processing Rules: action `content_view` sets `EVENT_CONTENT_VIEW`; each `content...` context data key sets its eVar.

=== "Google Analytics 4 (GTM)"

    | Item | Setup |
    |---|---|
    | Variables | `DLV - contentName` and so on, one per key |
    | Trigger | `CE - content_view` |
    | Tag | `GA4 - Content View`, event `content_view`, parameters `content_name`, `content_type`, `content_topic`, ... |
    | GA4 Admin | Register each parameter you want to report on as a custom dimension. Skip high-cardinality ones, such as `content_name`, if you do not need them as a dimension |

## QA checklist
- [ ] Fires on content pages only
- [ ] Every key is present, with empty text where it does not apply
- [ ] Values match the agreed lists exactly
- [ ] Multi-value keys use a comma and a space

## Notes
Publication date and reading time are text, not numbers. Keep the format fixed so reports sort and group correctly.
