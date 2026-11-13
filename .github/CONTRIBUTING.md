# Contributing to DP-700 Study Notes

Thanks for helping keep these notes accurate. They cover **Microsoft Exam DP-700: Implementing Data Engineering Solutions Using Microsoft Fabric**, and Microsoft Fabric ships changes almost weekly, so features get renamed, change their defaults or reach general availability between one revision of the study guide and the next. Reports from people studying with the notes are the best way to catch that.

> [!IMPORTANT]
> **This repo accepts issues, not pull requests.** Pull requests are turned off: I write every change myself and check it against the official documentation, so the notes keep one voice and stay consistent from page to page. If you know exactly how something should read, put your proposed wording in the issue. It's just as useful.

---

## Contents

- [Ways to Help](#-ways-to-help)
- [Before You Open an Issue](#-before-you-open-an-issue)
- [Opening an Issue](#-opening-an-issue)
- [Out of Scope](#-out-of-scope)
- [What Happens Next](#-what-happens-next)
- [Code of Conduct](#-code-of-conduct)
- [Security](#-security)

## 🙌 Ways to Help

Every report fits one of the four types in the issue template:

| Type | What it covers | For example |
|------|----------------|-------------|
| 🐞 **Bug** | Something on the site doesn't work: a broken link, search, navigation or a diagram that won't render | A Mermaid mind map shows as plain text, a link in a scenario table returns 404, or search can't find a heading that's on the page |
| ✏️ **Content issue** | Text, a table, a callout, a diagram or a code sample is wrong, unclear or misleading | A PySpark or KQL sample doesn't run as written, an exam caveat contradicts the docs, or a scenario table picks the wrong Fabric item for the requirement |
| 🕒 **Outdated content** | Microsoft Fabric or the DP-700 exam has changed since the page was written | A feature was renamed or reached general availability, a default changed (as V-Order's did), or the study guide added or reworded a skill |
| 📱 **Device issue** | A page looks or behaves wrong on a specific device, browser or screen size | A wide comparison table can't be scrolled sideways on a phone, or the sidebar menu won't open on a tablet |

## 🔎 Before You Open an Issue

- **Search the existing issues**, [open and closed](https://github.com/marcogrimaldi29/dp-700-study-notes/issues?q=is%3Aissue). If yours is already there, add a 👍 reaction or a comment with anything new rather than opening a duplicate.
- **Check which site you're on.** These are the v1 notes. The redesigned [DP-700 Study Notes v2](https://marcogrimaldi29.com/dp-700-study-notes-v2/) has its own repo, so please report anything on the v2 site in [its issues](https://github.com/marcogrimaldi29/dp-700-study-notes-v2/issues) instead.
- **Check the date.** The notes are written against the study guide's **skills measured as of October 19, 2026**. The **Skills measured** section of the [study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/dp-700) doesn't show a date at the moment, so look at the **Last updated on** date at the bottom of the page. If it's later than October 19, 2026, the skills list may have changed: compare it with the notes and mention any difference in the issue. It tells me which revision to compare against.
- **Check the current docs.** A quick look at [What's new in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/whats-new) or the [Microsoft Fabric documentation](https://learn.microsoft.com/en-us/fabric/) usually confirms whether something has changed, and gives you the link the issue asks for.
- **One problem per issue.** Separate issues are easier to track, fix and close. Several typos on the same page can go in one issue.
- **For a site bug, hard-refresh first** (Ctrl+F5, or Cmd+Shift+R on a Mac) in case your browser is showing an older copy of the page.

## 📝 Opening an Issue

1. Open a new issue with the [🐞 Report a problem with the notes](https://github.com/marcogrimaldi29/dp-700-study-notes/issues/new?template=ISSUE_TEMPLATE.md) template.
2. Give it a title that names the page and the problem, for example *"02 — Ingest & Transform: mirroring sources table is missing a source"* rather than *"Mistake on page"*.
3. Fill in the template's sections. Delete any that don't apply.

> [!TIP]
> To link straight to a section, hover over its heading and click the link icon that appears beside it, then copy the address from your browser.

### What Makes a Report Easy to Fix

- **A link to the exact section**, not just the page
- **The text as the notes have it**, quoted, so it can be found even if the page has moved on
- **An official source** for any content change: Microsoft Learn, the Fabric release notes or the DP-700 study guide. A report without one may take longer, because every change has to be checked against the documentation before it goes in
- **For bugs and device issues:** the steps to reproduce it, what you expected and what happened instead, the device, operating system and browser, and a screenshot if it shows the problem
- **Your proposed wording**, if you have it. With pull requests turned off, this is the closest thing to suggesting an edit directly

## 🚫 Out of Scope

To keep the notes accurate and the issue tracker useful, please don't open issues for:

- **Live exam content.** Questions, answers or topics remembered from a real exam break [Microsoft's exam security policy](https://learn.microsoft.com/en-us/credentials/support/exam-and-assessment-lab-security-policies). Issues or comments that include them will be closed and the content removed.
- **Exam booking, scores or retakes.** [Microsoft Credentials support](https://learn.microsoft.com/en-us/credentials/support/help) can help with those.
- **Help with your own Fabric tenant or workspace.** For troubleshooting a pipeline, a notebook, a capacity or your own code, ask in the [Microsoft Fabric Community](https://community.fabric.microsoft.com/) or on [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/), or contact [Microsoft Fabric support](https://support.fabric.microsoft.com/).
- **Anything private.** Issues are public, so leave out tenant and workspace IDs, connection strings, storage account keys, SAS tokens, service principal secrets and customer data, including in screenshots.
- **Unofficial sources.** Exam dump sites, unofficial practice tests and AI chat answers aren't accepted as the source for a content change.
- **Promotion.** Links to courses, products or services, affiliate links and paid placements won't be added.
- **Security vulnerabilities.** Please [report these privately](https://github.com/marcogrimaldi29/dp-700-study-notes/security/advisories/new), as described in the [security policy](SECURITY.md), not in a public issue.

## 🔄 What Happens Next

- I maintain the notes on my own, in my spare time, so replies are best effort and there's no fixed response time.
- I check each report against the official documentation. If something needs clarifying, I'll ask on the issue, so keep an eye on your GitHub notifications.
- When a fix goes in, the issue is closed with a note or a link to the change. The live site updates shortly after.
- If nothing changes, the issue is closed with the reason, such as out of scope, can't be verified, or working as intended.
- An issue waiting on a reply may be closed after a while. Comment with the missing details and it can be reopened.

## 🤝 Code of Conduct

By opening, commenting on or reacting to issues here, you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md). Comments that are abusive, spam or share live exam content may be hidden, edited or locked.

## 🔒 Security

Found a security problem with the site or this repo? Please don't report it in a public issue. Use [Report a vulnerability](https://github.com/marcogrimaldi29/dp-700-study-notes/security/advisories/new) instead, so only you and I can see it. The [security policy](SECURITY.md) has the details.

---

If the notes helped you, the README's [💚 Support](https://github.com/marcogrimaldi29/dp-700-study-notes#-support) section has a couple of ways to show it. Thanks for helping make them better for the next DP-700 candidate.
