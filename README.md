# Meta APIs, Rewritten for Developers

**Erica Nadia — Technical Writing Portfolio**

[![Deploy Portfolio](https://github.com/EricaNadia/technical-writing-portfolio/actions/workflows/deploy.yml/badge.svg)](https://github.com/EricaNadia/technical-writing-portfolio/actions/workflows/deploy.yml)

This portfolio demonstrates how documentation architecture reduces integration time, failure rate, and support burden for widely used Meta APIs — without replacing the official reference.

**Live site:** https://ericanadia.github.io/technical-writing-portfolio/

---

## What's here

Three production-grade guides, each built around a documented structural failure in Meta's developer ecosystem.

| Guide | Problem solved | Observed result |
|---|---|---|
| [WhatsApp Quick Start](https://ericanadia.github.io/technical-writing-portfolio/whatsapp-quickstart/) | Setup scattered across dashboards with no critical path | Setup time reduced from ~3 hours to ~15 minutes |
| [Graph API Error Reference](https://ericanadia.github.io/technical-writing-portfolio/graph-api-errors/) | Numeric error codes with no root cause or fix | Debugging time reduced from ~2 hours to ~2 minutes |
| [Permissions Guide](https://ericanadia.github.io/technical-writing-portfolio/permissions-guide/) | Conflated scope selection and App Review process | First-pass approval rate modeled at ~89% vs. ~58% baseline |

Results are from timed personal walkthroughs against Meta's official documentation and these rewritten guides, run across multiple iterations in clean environments. The App Review figure is modeled from documented rejection criteria, not a controlled study. See the homepage for full methodology.

---

## How the guides were built

- **Research:** Support forum threads, App Review rejection reports, and Stack Overflow questions — not happy-path assumptions
- **Testing:** All code verified against live Meta API v24.0 endpoints using registered WhatsApp Business accounts
- **Architecture:** Each guide has an explicit scope statement, progressive disclosure zones, and cross-references — not a flat list of instructions

---

## Documentation principles

Each principle maps to a concrete structural decision in the guides.

| Principle | Decision it drives |
|---|---|
| Start with a working outcome | Quick Starts end at a verified API call, not at "you're configured" |
| Surface irreversible decisions early | Destructive steps appear before setup instructions begin |
| Treat errors as first-class content | Error docs are structured by root cause, not numeric code |
| Separate platform constraints from developer mistakes | Every error entry classifies whether the fix is in code or requires a platform action |
| Scope explicitly | Every guide states what it covers and what it does not |
| Test before publishing | Untested examples are not included |

---

## Repository structure

```
.
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Pages CI/CD
├── docs/
│   ├── stylesheets/
│   │   └── custom.css          # Stripe-inspired theme overrides
│   ├── .nojekyll               # Disables Jekyll; required for MkDocs on GitHub Pages
│   ├── index.md                # Portfolio homepage
│   ├── whatsapp-quickstart.md
│   ├── graph-api-errors.md
│   └── permissions-guide.md
├── .gitignore
├── mkdocs.yml
└── README.md
```

---

## Stack and scope

**Languages / tools:** Python, JavaScript, cURL, Git, Postman  
**API surface:** WhatsApp Cloud API, Facebook Graph API, Meta authentication and permissions model  
**API version:** v24.0 (Feb 2026)  
**Build:** MkDocs Material — `mkdocs build --strict`

---

## Contact

📧 [erica.avitzur@gmail.com](mailto:erica.avitzur@gmail.com)  
💼 [LinkedIn](https://linkedin.com/in/ericaavitzur)  
🐙 [GitHub](https://github.com/EricaNadia)

---

*All timing metrics from personal walkthroughs against Meta's official documentation and these rewritten guides. App Review figure modeled from documented rejection criteria. Code verified against Meta API v24.0 (February 2026). Not affiliated with Meta Platforms, Inc.*
