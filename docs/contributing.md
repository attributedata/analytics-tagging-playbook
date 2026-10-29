# How to contribute

Anyone on the team can add or change a page. You do not need to install anything.

## Fix or improve an existing page (2 minutes)
1. Open the page on the site.
2. Click the **pencil icon** at the top right of the page.
3. Change the text, then click **Commit changes** and choose **Create a new branch and start a pull request**.
4. Write one line saying what you changed. Submit.

## Add a new topic (for example, e-commerce)
1. Open [`docs/tags/` on GitHub](https://github.com/attributedata/analytics-tagging-playbook/tree/main/docs/tags).
2. Click **Add file > Create new file**. Name it in lower case with hyphens, for example `ecommerce.md`.
3. Open `templates/tag.md`, copy everything, and paste it into your new file.
4. In the line under the title, keep the status as **Draft** and fill in your name and today's date, for example `29-Oct-2026`.
5. Fill in what you know. **Delete any section you cannot fill, but keep "Variables required" and "Developer specification".** Rough is fine; reviewers will help.
6. Add one line to the menu in `mkdocs.yml`, under the section that fits (see the table below).
7. Add a row for it to the "where to start" table on the [home page](index.md), with the same counts as its "Variables required" section.
8. Submit as a pull request.

### Which menu section?

| Section | For |
|---|---|
| Setup | Things developers install or build once: the tag manager, the data layer |
| Analytics Config | Settings made in the analytics tool itself, with little or no developer work |
| Core tags | Tracking for something visitors do or see: pages, search, forms, clicks, downloads, errors, video |
| Supporting config | Techniques that support other tags, such as event serialization |
| Testing | QA and debugging |

## What a good page contains
- **A developer specification:** the data layer push (JSON), when it fires, what each key holds with example values. A developer should be able to build it from this alone.
- **Tagging setup** for each implementation type: the data elements, rules and variable mappings. Write custom code only if a rule cannot do it.
- The number of eVars, props and events (and Google Analytics custom dimensions and events).
- A QA checklist.

## What a reviewer checks
- It says how many variables and events are needed, and the home page shows the same counts.
- It uses placeholders, not real eVar or event numbers.
- Keys and event names are the same across platforms.
- It was tested once, and the QA section says how.
- No client names or personal data. Use made-up example values.

## What happens next
- A lead reviews the pull request and may ask for changes.
- Once merged, the page publishes to the site automatically, marked **Draft**.
- When it has been tested on a real project, a lead changes its status from **Draft** to **Approved**.

## Page statuses
| Status | Meaning |
|---|---|
| Draft | Someone wrote it; nobody senior has reviewed and tested it yet |
| Approved | Reviewed and tested; use it on projects |
| Deprecated | Don't use for new work; the page says what replaced it |

## Writing tips
- Write for a new colleague who has never seen the client's setup.
- Show a good and a bad example of every value.
- Keep custom code short and say what each part does.
- Link to other pages instead of repeating them.
- When a spec changes, say so in the page's Notes, as a changelog line with date and name.
