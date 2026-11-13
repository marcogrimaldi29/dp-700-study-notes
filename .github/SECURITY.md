# Security Policy

DP-700 Study Notes is a static site built with Jekyll and the Just the Docs theme, and published to GitHub Pages. It has no accounts and no sign-in, and it sets no cookies and stores nothing in your browser. Search runs entirely in your browser, against an index that's downloaded with the site. Diagrams are drawn in your browser by Mermaid, loaded from the jsDelivr CDN, and analytics are cookieless, through [Umami](https://umami.is/). The attack surface is small, but security reports are still welcome, and I'd rather hear about a problem privately before it's public.

---

## 🛡️ Supported Versions

There are no releases. Only what's live is maintained.

| Version | Supported |
|---------|-----------|
| The live site at [marcogrimaldi29.com/dp-700-study-notes](https://marcogrimaldi29.com/dp-700-study-notes/) and the `main` branch | ✅ |
| Older commits, forks and copies hosted elsewhere | ❌ |

The redesigned [DP-700 Study Notes v2](https://marcogrimaldi29.com/dp-700-study-notes-v2/) is a separate repo, so please use [its private report form](https://github.com/marcogrimaldi29/dp-700-study-notes-v2/security/advisories/new) for anything on the v2 site.

## 📮 Reporting a Vulnerability

**Please don't open a public issue or comment.** Instead, report it privately through GitHub: open the repo's **Security** tab and choose **Report a vulnerability**, or go straight to the [private report form](https://github.com/marcogrimaldi29/dp-700-study-notes/security/advisories/new). Only you and I can see the report, and we can discuss and fix the problem there before anything is made public.

It helps to include:

- The affected URL or file in this repo
- What the problem is and what someone could do with it
- The steps to reproduce it, or a short proof of concept

Please don't include real secrets, credentials or anyone else's personal data in the report, and don't test against anything that isn't yours.

If you can't use GitHub, send me a private message on [LinkedIn](https://www.linkedin.com/in/marco-grimaldi29/) instead, starting it with **"Security: DP-700 Study Notes"** so it's easy to spot.

## 🎯 Scope

**In scope:**

- The published site, including its pages, scripts and assets
- This repository's source code and its GitHub Actions workflow
- A dependency version used here that has a known vulnerability

**Out of scope:**

- **Microsoft Fabric, Azure or any other Microsoft product.** Report these to the [Microsoft Security Response Center](https://msrc.microsoft.com/create-report).
- **A vulnerability in Jekyll, Just the Docs, Mermaid, Lunr or another dependency itself.** Report it to that project. If this repo uses an affected version, let me know as well.
- **HTTP response headers set by GitHub Pages.** The repo can't change them.
- **Automated scanner output** with no demonstrated impact on this site.
- **An insecure pattern in the notes**, such as a notebook sample that hard-codes a storage account key where a workspace identity or Azure Key Vault would be safer. That's a content issue, so it's welcome as a public issue — see the [contributing guide](CONTRIBUTING.md).

## ⏱️ What to Expect

I maintain this repo on my own, so responses are best effort. I'll confirm I've received your report, keep you posted in the private report while I look into it, and let you know when it's fixed. If you'd like to be credited, say so and I'll credit you in the security advisory or the fix.

Dependabot alerts and security updates are turned on for this repo, so known vulnerabilities in its dependencies are usually flagged and patched without a report.
