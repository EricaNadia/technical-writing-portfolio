# Meta APIs, Rewritten for Developers

Incorrect documentation decisions on large API platforms create operational cost at scale: engineers abandon integrations, support queues grow, and onboarding becomes a manual process. This portfolio demonstrates how documentation architecture — not documentation quality alone — can eliminate those failure modes measurably.

**I'm Erica Nadia.** I design API documentation for production systems where the cost of getting it wrong is visible in support tickets, integration drop-off, and failed App Reviews. My work is versioned, tested against live endpoints, and scoped to remain correct as platforms evolve.

---

## Measured impact

| Problem area | Before | After | Basis |
|---|---|---|---|
| WhatsApp API onboarding | ~3 hours | ~15 minutes | Timed walkthroughs |
| Debugging Graph API errors | ~2 hours | ~2 minutes | Timed walkthroughs |
| App Review first-pass approval | ~58% | ~89% (modeled) | Documented rejection criteria |

### Measurement methodology

Timing figures are derived from end-to-end walkthroughs performed against both Meta's official documentation and these rewritten guides. Each walkthrough used a clean environment, excluded account approval wait times, and measured elapsed time from first dashboard interaction to a successful API call or resolved error. Results reflect median times across multiple personal runs — not a cohort study, and not best-case execution.

The App Review approval figure is modeled from documented rejection patterns matched against the checklist criteria in the Permissions Guide. It is not a controlled study and is presented separately from the timing figures for that reason. All code was verified against live Meta API v24.0 endpoints using registered WhatsApp Business accounts.

---

## Documentation principles — and the decisions they drive

Each principle maps to a specific architectural choice made across this portfolio.

| Principle | Decision it drives |
|---|---|
| Start with a working outcome | Quick Starts terminate at a verified API call — not at "you're now configured" |
| Surface irreversible decisions early | Destructive steps (phone number registration, scope selection) appear before setup instructions begin |
| Treat errors as first-class content | Error docs are structured by root cause and developer intent, not by numeric code |
| Separate platform constraints from developer mistakes | Every error entry explicitly classifies whether the fix is in the developer's control or requires a platform action |
| Scope explicitly | Every guide opens with what it covers and what it does not — preventing misread responsibility and reducing support surface |
| Test before publishing | All code examples are verified against live endpoints. Untested examples are not included. |

*Documentation philosophy draws on Stripe's approach to outcome-first, error-aware reference design. See [Stripe documentation](https://stripe.com/docs) for context.*

---

## Case studies

### WhatsApp Business API: Quick Start

**Structural failure in the original ecosystem**
Meta's setup path is distributed across the developer dashboard, business verification pages, policy documentation, and API reference — with no single route from account creation to a working message. Average onboarding time: 2–3 hours, with significant drop-off at token configuration and business verification.

**What the rewrite does differently**

- Forces the test-vs-production decision before any action is taken — the wrong choice requires starting over with a new phone number
- Surfaces the six most common first-30-minute failures before Step 1 — knowing the pitfalls before encountering them changes how developers read setup instructions
- Terminates the Quick Start at Step 3 with an explicit finish line — remaining content is marked optional, not mandatory, for test-mode continuation
- Separates production setup into its own zone with its own checklist, so it is not confused with test setup

**Architectural decision**
Four visually distinct document zones — `## Quick Start`, `## Messaging context`, `## Production readiness`, `## Error reference` — communicate scope to a reader who may land mid-document. Each zone carries a different implied urgency. This reduces cognitive load under time pressure and allows the document to serve multiple reader states without restructuring.

[View the guide →](whatsapp-quickstart.md)

---

### Graph API Error Reference

**Structural failure in the original ecosystem**
Meta's error documentation maps numeric codes to descriptions. It does not explain root cause, does not distinguish between developer mistakes and platform-enforced behavior, and does not provide working fixes. Average debugging time: 2+ hours, with engineers frequently misattributing platform constraints to code bugs.

**What the rewrite does differently**

- Opens with a diagnostic model: errors are categorized as developer-controlled or platform-enforced before any code is shown
- Every error entry answers three questions: what caused this, is it fixable in code, and how do you prevent it recurring
- Prevention patterns are production-grade Python functions designed to eliminate error classes at the source — not code snippets
- Pre-flight validation section precedes all error entries — it prevents error classes from occurring rather than explaining them after the fact

**Architectural decision**
The `## Pre-flight validation` section is positioned before all error content. This mirrors defensive programming: validate at the boundary before the error can occur. It is a documentation architecture decision, not a content organization choice — the position of that section changes what a developer does before sending any request.

[View the reference →](graph-api-errors.md)

---

### WhatsApp Permissions Guide

**Structural failure in the original ecosystem**
Meta's permissions documentation conflates two separate problems: selecting the correct scopes, and proving to a reviewer that the use case justifies them. Developers who qualify for Standard Access are not told they can skip App Review entirely. Rejection reasons are vague. Consequence: a ~58% first-pass rejection rate and multi-week calendar delays.

**What the rewrite does differently**

- Immediately surfaces the most common case — Standard Access requires no review — so the majority of developers stop reading before the review section
- Separates the two problems explicitly: scope selection and review submission are distinct sections with distinct audiences
- Provides rejection recovery guidance with specific step-by-step instructions — not just submission, but what to do when it fails
- Cross-references the Quick Start and Error Reference so all three documents function as a navigable system

**Architectural decision**
The guide's `## Scope` section explicitly names what is out of scope and where to find it. This prevents a developer from using a permissions guide to debug token errors, or a quick start to handle review rejections. Scope discipline is an architectural choice that reduces support surface — readers who follow the wrong guide to the wrong answer still produce support cost.

[View the guide →](permissions-guide.md)

---

## Organizational impact

**Documentation that lets developers diagnose failures themselves — before opening a support ticket —** reduces reactive support load. When a developer can identify the root cause of Error 190 from the documentation before opening a ticket, the support queue shrinks — and the support team handles genuinely novel problems instead of repeating the same diagnosis.

**Explicit early decisions reduce onboarding variance.** When irreversible steps are surfaced before setup begins, the gap between a first-time integrator and an experienced one narrows. Onboarding timelines become predictable enough to plan against, not just buffer around.

**Scope discipline reduces institutional knowledge dependency.** Platform constraints documented in durable, versioned guides are available to the whole team — not just the engineer who debugged it once and remembers. This matters at platform scale, where the cost of undocumented constraints compounds across every new integrator.

**Tested, scoped documentation increases release confidence.** When every code example is verified against a live endpoint and every guide declares what it does not cover, the surface area for "documentation-caused failures" shrinks. Teams can ship knowing that the documentation is a reliable signal, not a source of uncertainty.

---

## How I de-risk developer platforms

### Failure-first research

I prioritize support forum threads, App Review rejection reports, and postmortems over happy-path flows. The goal is to locate decisions that are expensive to reverse when made incorrectly — and document those explicitly, before the developer reaches the step where the mistake becomes irreversible.

### Decision compression

I compress multi-dashboard, multi-policy ambiguity into explicit early decision points. The test-vs-production fork in the Quick Start and the Standard-vs-Advanced access table in the Permissions Guide are both examples — they make the cost of the wrong choice visible before it becomes irreversible.

### Verification discipline

Documentation is tested against live endpoints, real accounts, and real error responses. All code in this portfolio runs against Meta API v24.0 in clean environments. If an example cannot be verified, it does not ship. This eliminates the category of "works on paper" errors that send developers to support forums.

### Maintenance-aware structure

Each guide covers one concern. Cross-references are explicit links, not embedded assumptions. When an API surface changes, breakage is isolated to the section that covers it — not propagated through shared assumptions. This reduces the ongoing cost of keeping documentation correct as platforms evolve.

### Knowing what not to document

Every guide in this portfolio has an explicit out-of-scope statement. The Quick Start does not cover webhooks, templates, rate limits, or interactive messages — not because those are unimportant, but because including them in the onboarding path increases time-to-first-success. Scope is a documentation architecture decision with measurable consequences.

---

## What this portfolio deliberately does not do

- **Reproduce Meta's reference documentation.** The goal is not completeness — it is reducing the operational cost of the most common failure patterns across three specific surfaces.
- **Abstract away platform constraints for simplicity.** Developers need to know when behavior is platform-enforced and unfixable in code. Hiding that distinction is a documentation failure — it produces the wrong debugging behavior.
- **Optimize for tutorial completeness over correctness.** A code example that gets a developer 80% of the way there generates support cost. Every example in this portfolio is verified against live endpoints or not included.
- **Treat errors as an afterthought.** Real production integrations encounter errors on the first request. Documentation that covers errors only at the end reflects a happy-path assumption that production does not share.

---

## Background

Senior technical writer specializing in API platform documentation where failure carries revenue, compliance, or operational risk. Work focuses on reducing integration cost through documentation architecture, failure modeling, and decision design. Experience spans cross-team platform surfaces, developer onboarding, and evolving API contracts.

**Stack:** Python, JavaScript, cURL, Git, Postman  
**API surface:** RESTful (Graph API), Webhooks

---

## Contact

📧 [erica.avitzur@gmail.com](mailto:erica.avitzur@gmail.com)  
💼 [LinkedIn](https://linkedin.com/in/ericaavitzur)  
🐙 [GitHub](https://github.com/EricaNadia)

---

*Timing metrics from personal walkthroughs against Meta's official documentation and these rewritten guides. App Review figure modeled from documented rejection criteria — not a controlled study. Code verified against Meta API v24.0 (February 2026). Not affiliated with Meta Platforms, Inc.*
