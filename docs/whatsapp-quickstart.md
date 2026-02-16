# WhatsApp Business API: Quick Start

**Send your first WhatsApp message in under 15 minutes.**

Meta's official setup documentation is scattered across dashboards, policy pages, and changelogs — most developers spend 2–3 hours on a setup that should take 15 minutes. This guide isolates the critical path.

**Who this is for:** Backend engineers shipping production WhatsApp integrations who need first-message success without trial-and-error. REST and basic auth concepts assumed.  
**API version:** v24.0 (Feb 2026) — check [Meta's changelog](https://developers.facebook.com/docs/graph-api/changelog)

> All code tested: Python 3.12.3 · Flask 3.0 · requests 2.31.0 · February 2026

---

## Choose your path first

**Read this before creating anything.** Phone numbers cannot be transferred between WhatsApp accounts once registered. The wrong choice means starting over.

```
Are you sending to real users?
        │
        ├─ No ──→  Test mode  — 5 minutes, no verification needed
        │
        └─ Yes ──→ Production — 1–3 business days, verification required
```

| | Test mode | Production |
|---|---|---|
| **Use when** | Building, prototyping, CI | Sending to real users |
| **Token type** | Temporary (expires in 24h) | System User (never expires) |
| **Recipients** | Pre-approved list only (up to 5) | Any WhatsApp number |
| **Templates required** | No | Yes, outside the 24-hour window |
| **Business verification** | Not required | Required before going live |
| **Setup time** | ~5 minutes | 1–3 business days |

!!! danger "Irreversible"
    Once a phone number is registered to a WhatsApp Business Account, it cannot be transferred to a different account. Use a dedicated number — never your personal number.

---

## Common pitfalls — read before you start

These are the errors developers hit most in the first 30 minutes. Reading this table once prevents most of them.

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Using WABA ID instead of Phone Number ID | Error 100 — "object does not exist" | Use the 15-digit Phone Number ID, not the 16-digit WABA ID |
| `Bearer` prefix missing | Error 190 — token invalid | Header must be `Authorization: Bearer YOUR_TOKEN` |
| Recipient not in test allowlist | Error 131030 | Add recipient in dashboard → Phone Numbers → Send and receive messages |
| Phone number in wrong format | Error 100 or silent failure | E.164 format: `+` + country code + number, no spaces or dashes |
| Messaging after the 24-hour window | `200 OK` but message never arrives | Use an approved template for first contact or after 24 hours |
| Short-lived token in production | Error 190 after ~24h | Switch to a System User token — it does not expire |

---

## Quick Start — 15 minutes

### Step 1: Create your app and get credentials

1. Go to [developers.facebook.com](https://developers.facebook.com) → **My Apps** → **Create App**
2. Select **Business** type
3. Add the **WhatsApp** product to the app
4. Open **WhatsApp** → **API Setup** in the left sidebar

From the API Setup page, copy these three values — you need all of them:

| Value | Where to find it | Used for |
|-------|-----------------|----------|
| **Phone Number ID** | API Setup → "From" dropdown | All API calls — not the WABA ID |
| **Access Token** | API Setup → "Temporary access token" | Auth header in test mode |
| **Test phone number** | API Setup → "From" field | Your sending number |

The API Setup page shows both a Phone Number ID (15 digits) and a WABA ID (16 digits). Use the Phone Number ID in all API calls — using the WABA ID returns Error 100.

---

### Step 2: Add a test recipient

Test mode only delivers to numbers you explicitly pre-approve — up to five.

1. Dashboard → **WhatsApp** → **API Setup** → **To** field → **Manage phone number list**
2. Add the recipient's phone number in E.164 format (`+15551234567`)
3. The recipient receives a WhatsApp verification code and must confirm

---

### Step 3: Send your first message

Replace the placeholder values, then run:

```bash
curl -X POST "https://graph.facebook.com/v24.0/PHONE_NUMBER_ID/messages" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "+15551234567",
    "type": "text",
    "text": {
      "body": "Hello from WhatsApp Cloud API"
    }
  }'
```

**Success — HTTP 200:**

```json
{
  "messaging_product": "whatsapp",
  "contacts": [{ "wa_id": "15551234567" }],
  "messages": [{ "id": "wamid.HBgN..." }]
}
```

`messages[0].id` confirms Meta's servers accepted the request. The message should appear on the recipient's WhatsApp within seconds.

**It didn't work?** Check the [Common pitfalls](#common-pitfalls-read-before-you-start) table above, or jump to [Error reference](#error-reference).

---

### ✅ Quick Start complete

You've successfully sent a WhatsApp message via the API.

Here's the state of your system right now:

- You have a working test environment connected to Meta's sandbox
- Your token is temporary — it expires in 24 hours and is only valid in test mode
- Your sending is limited to the recipients you approved in Step 2
- No business verification, templates, or webhooks are required to continue in test mode

**What to read next depends on what you're building.**

The sections below are not required to continue in test mode. Come back to them when you're ready.

---

## Messaging context

The two most common sources of silent failures in WhatsApp integrations. Read before you build anything that sends messages to users.

### The 24-hour window

Outside the 24-hour window, the API returns `200 OK` with a message ID. The message is never delivered. No error is thrown. This is by design — Meta does not surface this as an API error.

**The rule:** Free-form messages are only allowed within 24 hours of the user's last inbound message to you. For all other cases — first contact, re-engagement, scheduled sends — use an approved template.

**Rule of thumb:** If you initiate contact, always use a template. If the user messaged first within the last 24 hours, free-form is allowed. When in doubt, use a template — they work in both states.

### Templates

Templates are required for first contact and all out-of-window sends. Meta provides a pre-approved `hello_world` template for testing.

**Send a template:**

```bash
curl -X POST "https://graph.facebook.com/v24.0/PHONE_NUMBER_ID/messages" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "+15551234567",
    "type": "template",
    "template": {
      "name": "hello_world",
      "language": { "code": "en_US" }
    }
  }'
```

**With variables** — for dynamic content like names or order numbers:

```json
{
  "type": "template",
  "template": {
    "name": "order_shipped",
    "language": { "code": "en_US" },
    "components": [{
      "type": "body",
      "parameters": [
        { "type": "text", "text": "Maria" },
        { "type": "text", "text": "ORDER-9981" }
      ]
    }]
  }
}
```

**Create a template:** Dashboard → **WhatsApp** → **Message Templates** → **Create Template**. Choose a category, write your content, submit. Approval typically takes minutes to a few hours.

#### Template categories

Template category affects review time, delivery rates, and cost. Getting this wrong is the leading cause of Error 368 (temporary block).

| Category | Use for | Examples |
|----------|---------|---------|
| **Authentication** | OTPs, login verification | "Your code is 123456" |
| **Utility** | Transactional, opt-in responses | Order confirmation, shipping update |
| **Marketing** | Promotions, offers, announcements | "New sale — 20% off this week" |

!!! warning "Marketing ≠ Utility"
    Submitting marketing content as Utility is the single most common cause of Error 368. If your template promotes a product, discount, or offer — it is Marketing, regardless of how you frame it.

### Webhooks

Webhooks deliver inbound messages, delivery receipts, and read status to your server. They are optional in test mode and required in production.

**Dashboard configuration:**

1. **WhatsApp** → **Configuration** → **Webhooks** → **Edit**
2. **Callback URL** — must be publicly reachable HTTPS. For local development, use [ngrok](https://ngrok.com): `ngrok http 5000`
3. **Verify token** — any string you choose, used to confirm endpoint ownership
4. Subscribe to the **messages** field → **Save**

**Verify endpoint ownership**

When you save the webhook, Meta sends a `GET` to confirm your server controls the URL:

```python
# Python 3.9+ | Flask 3.0 | Tested: Python 3.12.3
import os
from flask import Flask, request

app = Flask(__name__)
VERIFY_TOKEN = os.environ.get("WHATSAPP_VERIFY_TOKEN", "your_verify_token")

@app.route("/webhook", methods=["GET"])
def verify_webhook():
    """Handle Meta's webhook verification challenge."""
    mode = request.args.get("hub.mode")
    token = request.args.get("hub.verify_token")
    challenge = request.args.get("hub.challenge")

    if mode == "subscribe" and token == VERIFY_TOKEN:
        return challenge, 200
    return "Forbidden", 403
```

**Receive messages with signature verification**

Always verify the `X-Hub-Signature-256` header. Any client can POST to a public URL — requests without a valid signature should be rejected immediately.

```python
# Python 3.9+ | Flask 3.0 | Tested: Python 3.12.3
import hmac
import hashlib
import os
from flask import Flask, request

app = Flask(__name__)
APP_SECRET = os.environ["WHATSAPP_APP_SECRET"]

@app.route("/webhook", methods=["POST"])
def receive_webhook():
    """Receive and verify incoming WhatsApp webhook events."""
    signature = request.headers.get("X-Hub-Signature-256", "")
    expected = "sha256=" + hmac.new(
        APP_SECRET.encode(), request.data, hashlib.sha256
    ).hexdigest()

    if not hmac.compare_digest(signature, expected):
        return "Forbidden", 403

    payload = request.get_json(silent=True) or {}

    for entry in payload.get("entry", []):
        for change in entry.get("changes", []):
            value = change.get("value", {})

            for msg in value.get("messages", []):
                sender = msg.get("from")
                body = msg.get("text", {}).get("body", "")
                print(f"Message from {sender}: {body}")

            for status in value.get("statuses", []):
                print(f"Status for {status['id']}: {status['status']}")

    return "OK", 200
```

---

## Production readiness

Complete every item in this section before sending to any number outside your test allowlist.

### Business verification

Required before sending to any number outside your test allowlist. This is the step most teams underestimate — start it before you begin building.

1. **Business Manager** → **Business Settings** → **Business Info** → **Start Verification**
2. Submit business documentation — typically legal registration or a utility bill (varies by country)
3. Allow 1–3 business days for approval

Verification cannot be expedited and blocks all production sends until complete.

### Production credentials

Test mode tokens expire after 24 hours. Production requires a non-expiring System User token.

1. **Business Manager** → **Business Settings** → **System Users** → **Add**
2. Set role to **Admin**
3. **Add Assets** → select your WhatsApp Business Account
4. **Generate New Token** → select both permissions:
   - `whatsapp_business_messaging`
   - `whatsapp_business_management`
5. Copy the token — it is shown only once
6. Store it in your environment — never in source code:

```bash
export WHATSAPP_TOKEN="EAAG..."
export WHATSAPP_PHONE_ID="123456789012345"
export WHATSAPP_APP_SECRET="your_app_secret"
```

!!! danger "Never hardcode credentials"
    A leaked System User token gives unrestricted API access to your WhatsApp account. Use environment variables, AWS Secrets Manager, HashiCorp Vault, or equivalent. Never commit secrets to source control.

### Production-ready sending

The curl commands above are fine for exploration. Production code needs connection pooling, automatic retries on transient failures, and explicit timeouts — without these, a single slow request can block a thread indefinitely and a transient 5xx will surface as an unhandled exception.

```python
# Python 3.9+ | Tested: Python 3.12.3
import os
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

_session = requests.Session()
_session.mount(
    "https://",
    HTTPAdapter(
        max_retries=Retry(
            total=3,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504],
            allowed_methods=["POST"],
        )
    ),
)

GRAPH_API_BASE = "https://graph.facebook.com/v24.0"


def send_text_message(to: str, body: str) -> dict:
    """
    Sends a WhatsApp text message with connection pooling and retry logic.

    Args:
        to: Recipient phone in E.164 format (e.g. "+15551234567").
        body: Message text.

    Returns:
        API response dict — check messages[0].id for the message ID.

    Raises:
        requests.exceptions.HTTPError: On 4xx/5xx after retries exhausted.
        requests.exceptions.Timeout: If the request exceeds 10 seconds.
    """
    response = _session.post(
        f"{GRAPH_API_BASE}/{os.environ['WHATSAPP_PHONE_ID']}/messages",
        headers={"Authorization": f"Bearer {os.environ['WHATSAPP_TOKEN']}"},
        json={
            "messaging_product": "whatsapp",
            "recipient_type": "individual",
            "to": to,
            "type": "text",
            "text": {"body": body},
        },
        timeout=10,
    )
    response.raise_for_status()
    return response.json()
```

### Production checklist

This checklist does not cover consent capture UX, CRM integration, or message sequencing logic.

- [ ] Business verification approved
- [ ] Production phone number registered — dedicated number, not personal
- [ ] Payment method added to Meta Business account
- [ ] System User token generated with `whatsapp_business_messaging` and `whatsapp_business_management`
- [ ] Token and secrets stored as environment variables — not in source code
- [ ] Required message templates approved with correct categories
- [ ] Webhook live with HTTPS and signature verification enabled
- [ ] Retry logic implemented for HTTP 429 and 5xx responses
- [ ] Quality rating monitored in Meta Business Suite → Insights
- [ ] Opt-in mechanism in place for all recipients
- [ ] Opt-out path included in all outbound message sequences

---

## Error reference

The errors below are the ones developers hit most often in this guide's flow. For root cause analysis, subcodes, and prevention patterns, see the [Graph API Error Reference](graph-api-errors.md).

### Error 131030 — Recipient not in allowed list

**When it happens:** Test mode only.

**Fix:** Dashboard → **Phone Numbers** → select your test number → **Send and receive messages** → add the number. The recipient must confirm via a WhatsApp verification message.

---

### Error 190 — Invalid access token

**Cause:** Token expired, missing `Bearer` prefix, or generated for a different app.

```bash
# Confirm the header format
Authorization: Bearer YOUR_TOKEN

# Inspect the token
curl "https://graph.facebook.com/debug_token?\
input_token=YOUR_TOKEN&access_token=APP_ID|APP_SECRET"
```

For production: switch to a System User token. See [Production credentials](#production-credentials). For subcodes and prevention, see the [Graph API Error Reference](graph-api-errors.md#error-190-invalid-oauth-access-token).

---

### Error 100 — Invalid parameter

**Cause:** Wrong ID type or a required field is missing. Most common: WABA ID where Phone Number ID is required.

```bash
# Get your Phone Number ID from your WABA
curl "https://graph.facebook.com/v24.0/WABA_ID/phone_numbers?fields=id,display_phone_number" \
  -H "Authorization: Bearer TOKEN"
```

For subcodes and prevention, see the [Graph API Error Reference](graph-api-errors.md#error-100-invalid-parameter).

---

### Error 131047 — Messaging window expired

**Cause:** 24-hour window closed. The API returns `200 OK` — message is silently dropped.

If your logs show `200 OK` but users report non-delivery, assume a closed window before debugging anything else.

**Fix:** Switch to an approved template. See [Templates](#templates). See also [The 24-hour window](#the-24-hour-window).

---

### Error 368 — Temporary block

**Cause:** Policy violation — missing opt-in, miscategorized template, or sustained spam reports.

**Fix:** Check quality rating in Meta Business Suite → WhatsApp Manager → Insights → Quality Rating. Block lifts in 24–48 hours. For prevention patterns, see the [Graph API Error Reference](graph-api-errors.md#error-368-temporary-block).

---

## Next steps

| What you want to do | Where to go |
|--------------------|-------------|
| Send images, documents, video | [Media messages](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/messages#media-object) |
| Add buttons and lists | [Interactive messages](https://developers.facebook.com/docs/whatsapp/guides/interactive-messages) |
| Template approval and variable syntax | [Message templates](https://developers.facebook.com/docs/whatsapp/message-templates) |
| Rate limits and throughput tiers | [Messaging limits](https://developers.facebook.com/docs/whatsapp/messaging-limits) |
| Debug API errors in depth | [Graph API Error Reference](graph-api-errors.md) |

---

All code tested: Python 3.12.3 · Flask 3.0 · requests 2.31.0 · WhatsApp Cloud API v24.0 · February 2026  
Not affiliated with Meta Platforms, Inc.

*Last updated: February 2026*
