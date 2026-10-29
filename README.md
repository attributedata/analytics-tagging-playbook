# Analytics Tagging Playbook

Our team's source of truth for Adobe Analytics (Adobe Launch) and Google Analytics 4 (Google Tag Manager) tagging. Written in Markdown, published as a website.

**To read:** open the site (set up GitHub Pages, see below).
**To contribute:** read `docs/contributing.md`. You can do it entirely from the browser.

## Run it on your computer (optional)

    pip install -r requirements.txt
    mkdocs serve        # opens http://127.0.0.1:8000, updates as you save
    mkdocs build --strict

## First-time setup (once)
1. The repo is [attributedata/analytics-tagging-playbook](https://github.com/attributedata/analytics-tagging-playbook). Keep it private.
2. Replace the `YOUR-ORG/tagging-leads` team name in `.github/CODEOWNERS` with your reviewers' GitHub team.
3. In GitHub: Settings > Branches > require a pull request and a code-owner review on `main`.
4. In GitHub: Settings > Pages > deploy from the `gh-pages` branch. Pages for a private repo needs a plan that supports it; to keep the site team-only, use GitHub Enterprise Cloud's private Pages, or host the built `site/` folder anywhere behind your company login.
