# QA and Debugging

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## Tools

=== "Adobe Analytics"

    | Tool | Use for |
    |---|---|
    | Adobe Experience Platform Debugger (browser extension) | See which Launch library loaded and what each hit contains |
    | Browser developer tools, *Network* tab | Filter `/b/ss/` for AppMeasurement hits and `/ee/` for Web SDK hits |
    | `_satellite.setDebug(true)` in the console | See which Launch rules fire and why |
    | `adobeDataLayer` and `adobeDataLayer.getState()` in the console | See the pushes and the combined values |
    | Assurance | Mobile SDK sessions |

=== "Google Analytics 4"

    | Tool | Use for |
    |---|---|
    | GTM Preview mode (Tag Assistant) | See the data layer events, variables and which tags fired |
    | GA4 DebugView (*Admin > DebugView*) | See events and parameters arrive in near real time |
    | `dataLayer` in the console | See the pushes |

## Before testing
- Use the **staging** library and a test report suite or property. Do not test on production data.
- Use a private window, or clear cookies.

## Standard test for every feature
1. The developer's push appears in the data layer, with every key.
2. The right rule or tag fires, once.
3. The hit contains the correct values and the correct events.
4. Refresh and repeat the action. Check nothing is counted twice.
5. No personal data appears in any request.