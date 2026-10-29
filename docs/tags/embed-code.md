# Library Installation

Status: **Draft** · Owner: **Parikshit Shinge** · Last reviewed: **29-Oct-2026**
{ .page-meta }

## What this covers and why
The first thing developers add to the site is the tag manager's script, on every page. This page is the instruction to give them. Use the **staging** code on test sites and the **production** code on the live site, and keep the codes in the client's own spec.

## Variables required
None.

## When to fire
On every page, as high in `<head>` as possible. Create the data layer array **before** the script, so pushes made early are not lost.

## Developer specification

=== "Adobe Launch"

    Paste in the `<head>`, as high as possible:

    ```html
    <script>window.adobeDataLayer = window.adobeDataLayer || [];</script>
    <script src="<LAUNCH EMBED URL>" async></script>
    ```

    Use the **staging** embed URL on staging and the **production** one on the live site. Both come from Launch (*Environments > Install*). Add the Adobe Client Data Layer extension in the property, so the library handles `adobeDataLayer`.

=== "Google Tag Manager"

    Paste in the `<head>`, as high as possible:

    ```html
    <!-- Google Tag Manager -->
    <script>window.dataLayer = window.dataLayer || [];</script>
    <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
    new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
    j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
    'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
    })(window,document,'script','dataLayer','GTM-XXXXXXX');</script>
    <!-- End Google Tag Manager -->
    ```

    Paste immediately after the opening `<body>` tag:

    ```html
    <!-- Google Tag Manager (noscript) -->
    <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXXX"
    height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
    <!-- End Google Tag Manager (noscript) -->
    ```

    Replace `GTM-XXXXXXX` with the container ID. For staging, GTM gives an environment-specific snippet with extra parameters (`gtm_auth`, `gtm_preview`). Copy it from *Admin > Environments > Get Snippet*.

=== "Mobile apps"

    The developers add the Adobe Experience Platform Mobile SDK and register the extensions with the app ID from the Launch mobile property, using the development, staging and production environment IDs.

## Tagging setup
Create one Launch property (or GTM container) per site, with development, staging and production environments. Publish to staging first and test (see [QA and Debugging](qa-debugging.md)).

## QA checklist
- [ ] The script is on **every** page, including error pages and the thank-you pages of forms
- [ ] It loads once only
- [ ] Staging code is not on the live site, and production code is not on staging
- [ ] The data layer array exists before the script

## Notes
Keep the real embed URLs and container IDs out of this playbook. Put them in the client's own spec document.
