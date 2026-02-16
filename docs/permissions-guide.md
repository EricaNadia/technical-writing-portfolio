---
description: WhatsApp Cloud API permissions guide — which scopes to request, whether you need App Review, how to pass it on the first submission, and how to recover from rejections.
---

# WhatsApp Permissions Guide

**Get App Review approval on the first try.**

Meta's permissions system has two distinct problems that most documentation conflates: deciding which permissions to request, and proving to a reviewer that your use case justifies them. This guide separates both. It tells you exactly what to request, why, and how to make the review process predictable.

**Who this is for:** Developers integrating WhatsApp Cloud API who need to understand permission scopes, token lifecycle, and App Review requirements. Assumes you have a Meta Developer account and a basic working app.  
**API version:** v24.0 (Feb 2026) — check [Meta's changelog](https://developers.facebook.com/docs/graph-api/changelog)

> All code tested: Python 3.12.3 · requests 2.31.0 · February 2026

---

## Scope

This guide covers:

- Which permissions WhatsApp integrations require and why
- Standard vs Advanced access — which path applies to your use case
- How to structure an App Review submission that passes on the first attempt
- What to do when a review is rejected
- Token debugging and permission verification

**Out of scope:** Embedded Signup implementation, webhook configuration, template approval, and rate limiting. See the [WhatsApp Quick Start](whatsapp-quickstart.md) and [Graph API Error Reference](graph-api-errors.md) for those topics.

---

## How to use this guide

1. Check [Which permissions you need](#which-permissions-you-need) — most integrations need exactly two.
2. Check [Access types](#access-types) — **Standard Access covers most use cases without any review.** If you qualify, stop here.
3. If you need Advanced Access, use the [App Review checklist](#app-review-checklist-advanced-access-only) before submitting.
4. If your review is rejected, see [Handling rejections](#handling-rejections).
5. For token issues in production, see [Debugging permissions](#debugging-permissions) and [Error 200](#error-200-permission-required).

---

## Which permissions you need

Most WhatsApp integrations require exactly two permissions. Requesting more than you use is the second most common cause of rejection — reviewers look for scope justification.

| Permission | Enables | Does not cover |
|---|---|---|
| `whatsapp_business_messaging` | Send/receive messages, media, webhooks, interactive messages | Managing numbers, creating templates, viewing analytics |
| `whatsapp_business_management` | Register numbers, create templates, view analytics, manage settings | Sending or receiving messages |

**Production apps need both.** Neither scope alone is sufficient for a complete integration: messaging without management means you cannot create or manage templates; management without messaging means you can configure everything but send nothing.

!!! warning "Request only what you use"
    Every additional scope you request requires its own justification video and increases review time. Request only `whatsapp_business_messaging` and `whatsapp_business_management`. Do not add scopes speculatively.

---

## Access types

The access type determines whether App Review is required at all. Most integrations qualify for Standard Access.

| Scenario | Access type | App Review required? | Typical timeline |
|---|---|---|---|
| Testing your own app | Standard | No | ~5 minutes |
| Production, single business (your own WABA) | Standard | No | 1–3 business days |
| SaaS / sending on behalf of other businesses | Advanced | Yes | 3–7 business days |
| Using Embedded Signup to onboard other businesses | Advanced | Yes | 3–7 business days |

!!! note "The most common mistake"
    Developers building for their own WABA submit App Review unnecessarily, then wait days for approval they didn't need. Check this table before starting any review process.

---

## Standard Access path

If you are building for your own WhatsApp Business Account, Standard Access is auto-granted. You do not need App Review. Standard Access is sufficient for production use when you control the WABA your app connects to.

**If Standard Access covers your use case, you can stop reading here.** The rest of this guide covers Advanced Access and App Review.

For production token setup, jump to [Production token setup](#production-token-setup).

---

## Advanced Access path

Advanced Access is required when your app acts on behalf of other businesses — for example, a SaaS platform that lets customers connect their own WABAs, or an Embedded Signup flow that onboards new WABA owners. Advanced Access requires App Review and business verification.

### Before you submit

If you do need Advanced Access, first-time approval requires specific materials. Incomplete or vague submissions are rejected without the opportunity to fix them before re-review.

#### What reviewers check

Every review evaluates three things:

1. **Legitimacy** — is this a real business with a real use case?
2. **Proportionality** — does the scope match what the app actually does?
3. **Compliance** — does the app collect opt-in, provide opt-out, and have a privacy policy that covers WhatsApp?

All rejection patterns trace back to one of these three checks failing. The checklist below is organized around them.

### App Review checklist {#app-review-checklist-advanced-access-only}

**Legitimacy**

- [ ] Business verification completed in Meta Business Manager
- [ ] App icon uploaded — 1024×1024px, no placeholder
- [ ] Privacy policy URL is publicly accessible (not localhost, not behind login)

**Proportionality**

- [ ] Use case explanation is specific, not vague — see [Rejection patterns](#rejection-patterns) for examples
- [ ] One short demo video per permission requested (30 seconds each, not combined)
- [ ] Working test credentials provided, or an unlisted Loom link showing the full flow

**Compliance**

- [ ] Privacy policy explicitly mentions WhatsApp data and opt-out
- [ ] Opt-in screenshot shows a signup form with a WhatsApp consent checkbox
- [ ] Opt-out process demonstrated — STOP keyword, unsubscribe link, or equivalent

---

## Rejection patterns

These are the five most common rejection reasons, with the exact correction for each.

### "Use case unclear"

Reviewers reject descriptions that could apply to any app. Describe the specific trigger, the specific message, and the specific recipient.

```
✗  "Send messages to customers"
✓  "Send order-shipped notifications with a tracking link immediately after
    a purchase confirms in our e-commerce platform"
```

### "No opt-in shown"

A generic screenshot of your app is not sufficient. The screenshot must show a form field where a user explicitly consents to WhatsApp messages.

```
✗  Generic app screenshot
✓  Signup form with visible checkbox: "[ ] Send me WhatsApp order updates"
```

### "Privacy policy doesn't mention WhatsApp"

A generic data policy does not satisfy this requirement. The policy must include a WhatsApp-specific section.

```
✗  Generic: "We collect data to improve your experience"
✓  Explicit: "We send order updates via WhatsApp. Reply STOP at any time to opt out.
    We do not share your number with third parties."
```

### "Multiple permissions in one video"

Each permission needs its own video demonstrating its use. A single video covering both permissions is rejected.

```
✗  One 3-minute demo covering everything
✓  Two 30-second clips — one showing a message send, one showing template management
```

### "Test credentials don't work"

If a reviewer cannot reproduce the flow, the review fails regardless of how well everything else is documented.

```
✗  "Use test@example.com / password123"
✓  Real working credentials, or an unlisted Loom recording of the complete flow
```

---

## Handling rejections

Rejection is not final. Every rejection includes a stated reason — respond to it exactly.

**Step 1 — Read the rejection reason literally.** Meta's rejection messages are specific. "Privacy policy doesn't mention WhatsApp" means add a WhatsApp section to your privacy policy — not rewrite the whole thing.

**Step 2 — Fix only what was cited.** Do not change unrejected materials. Changing correct materials introduces new rejection risk.

**Step 3 — Re-submit with a note.** The re-submission form includes a message field. State explicitly what you changed and why it addresses the rejection: *"Added WhatsApp opt-out section to privacy policy at [URL]. Screenshot attached."*

**Step 4 — If the same reason recurs.** A repeated rejection on the same point usually means the fix didn't fully satisfy the criterion. Check the [Rejection patterns](#rejection-patterns) section above and compare your submission against the ✓ examples exactly.

!!! warning "Re-review timelines reset"
    Each re-submission restarts the review clock (3–7 business days). Fix everything before re-submitting — partial fixes that require a third submission cost weeks.

---

## Error 200 — Permission required

```json
{
  "error": {
    "code": 200,
    "message": "Requires whatsapp_business_messaging permission"
  }
}
```

This error means the token in use does not have the required scope. It is almost always a token configuration issue — not an App Review or access type issue.

**The most important thing to understand:** App Review approval does not add scopes to existing tokens. After approval, you must regenerate your token with the newly approved scopes selected. See the [Graph API Error Reference](graph-api-errors.md#error-200-permission-required) for the full breakdown including a startup validation pattern.

### Quick fix

Check what scopes the token actually has:

```bash
curl "https://graph.facebook.com/debug_token?\
input_token=YOUR_TOKEN&\
access_token=APP_ID|APP_SECRET"
```

```json
{
  "data": {
    "is_valid": true,
    "scopes": ["whatsapp_business_management"]
  }
}
```

If `whatsapp_business_messaging` is missing, regenerate the token with both scopes selected. The token must be regenerated — existing tokens are not updated by scope changes.

---

## Production token setup

System User tokens do not expire unless manually revoked. Use them for all server-side messaging. Short-lived user tokens are appropriate only for local development and testing.

**Generate a System User token:**

1. Business Manager → **Business Settings** → **System Users** → **Add**
2. Set role to **Admin**
3. **Add Assets** → select your WhatsApp Business Account
4. **Generate New Token** → select `whatsapp_business_messaging` and `whatsapp_business_management`
5. Copy the token — it is shown only once

**Store it in your environment — never in source code:**

```bash
export WHATSAPP_TOKEN="EAAG..."
export WHATSAPP_PHONE_ID="123456789012345"
```

**Validate at startup** — this pattern catches a missing or misconfigured token before any request reaches the API:

```python
# Python 3.9+ | Tested: Python 3.12.3
import os
import requests

REQUIRED_SCOPES = {
    "whatsapp_business_messaging",
    "whatsapp_business_management",
}

def validate_token_scopes(token: str, app_id: str, app_secret: str) -> None:
    """
    Confirms the token has all required scopes.
    Call once at application startup — not per request.
    Raises PermissionError if scopes are missing or the token is invalid.
    """
    try:
        response = requests.get(
            "https://graph.facebook.com/debug_token",
            params={
                "input_token": token,
                "access_token": f"{app_id}|{app_secret}",
            },
            timeout=10,
        )
        response.raise_for_status()
    except requests.exceptions.RequestException as exc:
        raise RuntimeError(f"Could not reach Meta token endpoint: {exc}") from exc

    data = response.json().get("data", {})

    if not data.get("is_valid"):
        raise PermissionError("Token is invalid. Regenerate before checking scopes.")

    missing = REQUIRED_SCOPES - set(data.get("scopes", []))
    if missing:
        raise PermissionError(
            f"Token is missing required scopes: {missing}. "
            "Regenerate the token and select all required permissions. "
            "App Review approval does not add scopes to existing tokens."
        )
```

!!! danger "Never hardcode tokens"
    A leaked System User token gives unrestricted API access to your WhatsApp account. Use environment variables, AWS Secrets Manager, or HashiCorp Vault. Never commit secrets to source control.

---

## Debugging permissions

### Check what a token has been granted

```bash
curl "https://graph.facebook.com/v24.0/me/permissions" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

```json
{
  "data": [
    { "permission": "whatsapp_business_messaging",  "status": "granted"  },
    { "permission": "whatsapp_business_management", "status": "declined" }
  ]
}
```

A `"declined"` status means the user explicitly denied that scope during token generation — not that App Review failed. Regenerate the token and select the scope.

### Verify token validity, expiry, and issuing app

```bash
curl "https://graph.facebook.com/debug_token?\
input_token=YOUR_TOKEN&\
access_token=APP_ID|APP_SECRET"
```

Key fields to check in the response:

| Field | What it tells you |
|-------|------------------|
| `is_valid` | Whether the token is currently usable |
| `expires_at` | Unix timestamp — `0` means non-expiring (System User token) |
| `scopes` | Array of scopes this token was issued with |
| `app_id` | Confirms the token belongs to the expected app |

If `is_valid` is `false`, the token must be regenerated before any other debugging step.

---

## Resources

| Resource | Use for |
|----------|---------|
| [Permissions Reference](https://developers.facebook.com/docs/permissions) | Full list of available scopes |
| [App Review Guide](https://developers.facebook.com/docs/app-review) | Official submission requirements |
| [WhatsApp Business Policy](https://www.whatsapp.com/legal/business-policy) | Acceptable use rules — read before submitting |
| [Token Debugger](https://developers.facebook.com/tools/debug/accesstoken) | Browser-based token inspection |
| [Graph API Explorer](https://developers.facebook.com/tools/explorer) | Live API calls in-browser |
| [API Changelog](https://developers.facebook.com/docs/whatsapp/changelog) | Breaking changes and deprecations |

**In this portfolio:**

- [WhatsApp Quick Start](whatsapp-quickstart.md) — first message in 15 minutes, test and production setup
- [Graph API Error Reference](graph-api-errors.md) — root causes and prevention patterns for the six most common errors

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| v1.2 | Feb 2026 | Restructured into Standard Access path and Advanced Access path with explicit stop points for each audience; moved App Review checklist under Advanced Access section; updated How to use section to reflect new structure |
| v1.1 | Feb 2026 | Added Scope section; How to use section; Handling rejections section; startup token validation pattern; debugging fields table; cross-references to portfolio docs; heading hierarchy corrected for MkDocs TOC |
| v1.0 | Feb 2026 | Initial release |

---

All code tested: Python 3.12.3 · requests 2.31.0 · WhatsApp Cloud API v24.0 · February 2026  
Not affiliated with Meta Platforms, Inc.

*Last updated: February 2026*
