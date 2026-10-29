# Video Tracking

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

!!! warning "Not tested yet"
    This tag started as a proposal and has not been tested on a live property. Check the open questions under [Notes](#notes) before you use it on a project.

## What this tracks and why
How many people start, finish and drop out of videos on the site.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | eVars | 2 | `EVAR_VIDEO_NAME`, `EVAR_VIDEO_MILESTONE` |
    | Props | 0 | |
    | Events | 5 | `EVENT_VIDEO_START`, `EVENT_VIDEO_25`, `EVENT_VIDEO_50`, `EVENT_VIDEO_75`, `EVENT_VIDEO_COMPLETE` |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 2 | Video Name, Video Milestone |
    | Events | 1 | `video_milestone`. Not `video_progress`: GA4's built-in YouTube tracking already uses `video_start`, `video_progress` and `video_complete` |

## When to fire
Start when the video plays for the first time. Then once at 25%, 50%, 75% and complete. Not again if the visitor replays or seeks back.

## Developer specification

=== "Adobe websites (adobeDataLayer)"

    ```javascript
    window.adobeDataLayer = window.adobeDataLayer || [];
    window.adobeDataLayer.push({
      'event': 'video_milestone',
      'videoName': '<video name>',
      'videoMilestone': '<0, 25, 50, 75 or 100>'
    });
    ```

=== "Google Analytics websites (dataLayer)"

    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'video_milestone',
      'videoName': '<video name>',
      'videoMilestone': '<0, 25, 50, 75 or 100>'
    });
    ```

=== "Mobile apps"

    ```swift
    // Consider Adobe's Media extension, which tracks playback for you.
    MobileCore.track(action: "video_milestone", data: ["videoName": "How to apply", "videoMilestone": "50"])
    ```

| Key | What it holds | Example |
|---|---|---|
| `videoName` | The video's title | `How to apply` |
| `videoMilestone` | Percent watched | `50` |

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    One rule `ACDL - Video: Milestone by "video_milestone"`, with five conditions or five rules (one per milestone), each setting its event and the two eVars.

=== "Adobe Web SDK (XDM)"

    Adobe has standard media events in XDM (for example `media.play`). Check Adobe's Media Analytics guidance before adding custom events.

=== "Adobe Mobile SDK"

    Consider the Media extension before building custom processing rules.

=== "Google Analytics 4 (GTM)"

    Trigger on `video_milestone`; GA4 Event tag with parameters `video_name`, `video_milestone`. Turn off enhanced measurement *Video engagement* if you track it yourself.

## QA checklist
- [ ] Each milestone fires once
- [ ] Seeking forward does not skip or duplicate milestones

## Notes
- 29-Oct-2026: moved from Proposals to Core tags as a Draft tag. Counts added to the totals.
- 29-Oct-2026: GA4 event name set to `video_milestone` (was `video_progress` in the Variables table), to match the data layer push and avoid GA4's built-in video events.
- Open question: use Adobe Media Analytics instead of custom events?
- Open question: which video players does each client use?
