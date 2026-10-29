# Marketing Channels

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this tracks and why
Which campaign, ad or email brought a visitor to the site. In Adobe, **Marketing Channels** reports sort visits into channels (Paid Search, Email, Display and so on) using rules in Admin, and the **tracking code** in the landing page URL names the exact campaign. In Google Analytics, UTM parameters do the same job.

## Variables required

=== "Adobe Analytics"

    | Type | How many | What each one holds |
    |---|---|---|
    | Campaign variable | 1 | Tracking code (`s.campaign`) |
    | eVars | 0 (1 optional) | `EVAR_TRACKING_CODE`: only if you need a different expiry or allocation |
    | Props | 0 | |
    | Events | 0 | |
    | Other | | Marketing Channel rules, set once per report suite in Admin |

=== "Google Analytics 4"

    | Type | How many | What each one holds |
    |---|---|---|
    | Custom dimensions | 0 | Uses built-in source, medium and campaign |
    | Events | 0 | |

## When to fire
On the **first page of a visit** whose URL has the tracking code. Not on reloads or later pages.

## Values and naming
Adobe tracking code: `channel_source_campaign_content`, lower case, in the URL parameter `cid`. Agree one parameter name for all clients.

| Good | Bad |
|---|---|
| `email_newsletter_autumn-promo_hero-button` | `Autumn Promo!!` |
| `ppc_google_cards-2026_brand` | `cards ppc` |

Google Analytics: use the standard `utm_source`, `utm_medium`, `utm_campaign` and `utm_content`. Do not put customer names or emails in either.

## Developer specification
**Nothing for web developers.** The marketing team adds the parameters to campaign links, and the tools read them from the URL. For internal promotions, do **not** use `cid` or UTM parameters (see [Universal Clickthrough](clickthrough.md)).

For **mobile apps**, the campaign usually arrives in a deep link. Developers read it and send it:

=== "Mobile apps"

    ```swift
    // iOS: when the app opens from a link
    func handleDeepLink(_ url: URL) {
        let cid = URLComponents(url: url, resolvingAgainstBaseURL: false)?
            .queryItems?.first(where: { $0.name == "cid" })?.value
        if let cid = cid {
            MobileCore.track(action: "campaign_open", data: ["trackingCode": cid])
        }
    }
    ```

    ```kotlin
    // Android
    val cid = intent.data?.getQueryParameter("cid")
    if (cid != null) MobileCore.trackAction("campaign_open", mapOf("trackingCode" to cid))
    ```

## Tagging setup

=== "Adobe Analytics (AppMeasurement)"

    | Data element | Type | Setting |
    |---|---|---|
    | `Query - cid` | Query String Parameter | Parameter `cid`, convert to lower case |

    | Rule | Trigger | Actions |
    |---|---|---|
    | `PLR - Global: Page View` (add to the existing rule) | Page Bottom | Set Variables: Campaign = `%Query - cid%` (and `EVAR_TRACKING_CODE` if used) |

    Custom code (only if needed): without it, a page refresh counts the code again. The Common Analytics Plugins extension provides `getValOnce`:

    ```javascript
    s.campaign = s.getValOnce(_satellite.getVar('Query - cid'), 's_cid', 0);
    ```

    Then, in Adobe Analytics, set the Marketing Channel rules (*Admin > Report Suites > Edit Settings > Marketing Channels > Marketing Channel Rules*). The first rule that matches wins, so order matters.

=== "Adobe Web SDK (XDM)"

    In the page-view XDM object, set `marketing.trackingCode` to `%Query - cid%` (check the field in your schema). Send it only on the first page of the visit. Marketing Channel rules are still set in Admin.

=== "Adobe Mobile SDK"

    Processing Rule: when action name equals `campaign_open`, set Campaign = context data `trackingCode`. Then set the Marketing Channel rules in Admin.

=== "Google Analytics 4 (GTM)"

    Nothing to build. The Google tag reads the UTM parameters and GA4 builds the Traffic acquisition reports. Check that:

    - The Google tag fires on every page, including the landing page, before any redirect drops the parameters.
    - Campaign links use one agreed UTM format.
    - Internal links never use UTM parameters.

    For custom channel groups, create them in GA4 *Admin > Data display > Channel groups*.

## QA checklist
- [ ] Visit `?cid=test_qa_check_1`: the code is sent once
- [ ] Refresh: it is not sent again
- [ ] Visit with no `cid`: nothing is sent
- [ ] The visit shows in the expected Marketing Channel (after processing)
- [ ] A redirect on the landing page does not drop the parameters

## Notes
Marketing Channels in Adobe need both the tracking code **and** the rules in Admin. The code alone does not create channels.
