# WhatsApp LLM Chatbot — Workflow Documentation

A complete guide to this n8n workflow: what it does, what you need before running it, how to configure every credential, and how to import and test it from scratch.

---

## What This Workflow Does

This workflow turns your self-hosted n8n instance into a **WhatsApp chatbot** powered by an LLM (Groq, running an OpenAI open-weight model). When someone messages your WhatsApp Business number, the message is received by n8n, sent to an AI model for a reply, and the AI's response is sent back to the user — all automatically.

**Flow at a glance:**

```
WhatsApp message sent by user
        ↓
Meta (WhatsApp Cloud API) forwards it to your n8n webhook
        ↓
n8n extracts the sender + message text
        ↓
Message is sent to Groq (AI model) for a reply
        ↓
n8n sends the AI's reply back to the user via WhatsApp
```

---

## What You Need Before Starting

You need accounts/keys from **two providers**, plus n8n running somewhere reachable from the internet.

| # | What you need | Where to get it | Cost |
|---|---|---|---|
| 1 | n8n installed & running | Self-hosted (see this repo's install guide) | Free |
| 2 | A public URL pointing to your n8n instance | ngrok (for testing) or a real domain (for production) | Free (ngrok) |
| 3 | Meta Developer App + WhatsApp Cloud API access | developers.facebook.com | Free (test tier) |
| 4 | WhatsApp test phone number + Access Token | Comes from the Meta App setup above | Free |
| 5 | Groq API key | console.groq.com → API Keys | Free tier available |

---

## Step-by-Step: A to Z

### Part 1 — Get n8n running

1. Install n8n locally (Docker recommended) — see [`install guide.md`](./install%20guide.md) in this repo if you haven't already.
2. Confirm it's reachable at `http://localhost:5678`.

### Part 2 — Expose n8n to the internet (required for WhatsApp)

Meta's servers need to reach your n8n webhook, so `localhost` alone won't work.

For testing, use **ngrok**:
```powershell
ngrok http 5678
```
This gives you a temporary public URL like `https://xxxx.ngrok-free.app`. Keep this terminal window open while testing — closing it kills the tunnel and changes the URL next time you start it.

> For production, replace ngrok with a real domain pointed at your server, behind HTTPS (see the "Production Recommendation" section in the install guide).

### Part 3 — Set up the Meta Developer App (WhatsApp Cloud API)

1. Go to **developers.facebook.com** → log in → **My Apps → Create App**
2. Choose **Business** as the app type → fill in name/email → Create
3. On the app dashboard, find **WhatsApp** under "Add products" → **Set Up**
4. Connect/create a **Meta Business Account** if prompted
5. On the **WhatsApp → API Setup** page, note down:
   - **Temporary Access Token** (valid 24 hours — you'll regenerate this often during testing)
   - **Phone Number ID** (also auto-extracted by the workflow from incoming messages, but good to have on hand)
6. Under "To" / recipient list, **add your own WhatsApp number** as a tester and verify the OTP sent to it.
7. Still on this page, configure the **Webhook**:
   - **Callback URL:** `https://xxxx.ngrok-free.app/webhook/whatsapp-webhook`
   - **Verify Token:** `myVerifyToken123` (must match exactly — see Part 5 below if you want to change this)
   - Click **Verify and Save**
   - Subscribe to the **`messages`** field

### Part 4 — Get a Groq API key

1. Go to **console.groq.com** → sign up/log in
2. Navigate to **API Keys** → **Create API Key**
3. Copy the key — you won't be able to see it again after leaving the page

### Part 5 — Import the workflow into n8n

1. Download `WhatsApp_LLM_Chatbot_fixed.json` from this repo
2. In n8n: click the **⋯ menu (top right)** → **Import from File** → select the JSON file
3. The workflow will appear on your canvas with all nodes and connections pre-wired

### Part 6 — Configure credentials in n8n

You need to set up **two credentials** inside n8n (the imported JSON references them by name, but you must create/link them yourself — credentials are never included in exported workflow files, for security).

**A. Groq credential**
1. Click the **Groq Chat Model** node
2. Under Credential, click **Create New** (or select existing if you already have one named "Groq account")
3. Paste your Groq API key → Save

**B. Meta WhatsApp Access Token credential**
1. Click the **Send WhatsApp Reply** node
2. Under Authentication, it's set to use an **HTTP Header Auth** credential — click **Create New**
3. Set:
   - **Name:** `Authorization`
   - **Value:** `Bearer YOUR_META_ACCESS_TOKEN` (paste your temporary or permanent token from Part 3)
4. Save

### Part 7 — (Optional) Change the verify token

The workflow ships with a placeholder verify token: `myVerifyToken123`. This is just a shared secret between you and Meta — you can leave it as-is or change it.

To change it:
1. Open the **Check Verify Token** node
2. Update the `rightValue` field to your own custom string
3. Make sure the **same value** is entered in the Meta App's Webhook configuration (Part 3, step 7)

### Part 8 — Activate the workflow

Toggle the **Active** switch (top-right of the n8n canvas) to ON. The workflow must be active for the webhook URLs to work — testing via "Listen for test event" only works for manual single-run tests, not live traffic.

---

## How to Test

### Test 1 — Webhook verification (confirms Meta can reach n8n)

```powershell
curl "https://xxxx.ngrok-free.app/webhook/whatsapp-webhook?hub.mode=subscribe&hub.verify_token=myVerifyToken123&hub.challenge=12345"
```
Expected response: `12345`

### Test 2 — Simulate an incoming WhatsApp message (no real phone needed)

```powershell
curl -X POST "https://xxxx.ngrok-free.app/webhook/whatsapp-webhook" -H "Content-Type: application/json" -d "{\"entry\":[{\"changes\":[{\"value\":{\"metadata\":{\"phone_number_id\":\"123456789\"},\"messages\":[{\"from\":\"923001234567\",\"text\":{\"body\":\"Hello there\"}}]}}]}]}"
```
Watch the n8n canvas — nodes should light up as the execution runs. Check the **Basic LLM Chain** node's output to confirm Groq generated a reply. (The final "Send WhatsApp Reply" step will fail here since `123456789` isn't a real phone number ID — that's expected.)

### Test 3 — Full real-world test

From your verified test WhatsApp number, send a message to your Meta test business number. Within a few seconds, you should receive an AI-generated reply back on WhatsApp.

Check the **Executions** tab in n8n (left sidebar) to see the full run and debug any failed nodes.

---

## Workflow Node Reference

| Node | Purpose |
|---|---|
| **Webhook Verify (GET)** | Handles Meta's one-time webhook verification handshake |
| **Check Verify Token** | Confirms the verify token in the request matches your configured secret |
| **Respond Challenge** | Returns Meta's challenge string back, completing verification |
| **Respond Forbidden** | Returns a 403 if the verify token doesn't match (security check) |
| **Webhook Incoming (POST)** | Receives every real incoming WhatsApp message from Meta |
| **Ack Meta** | Immediately acknowledges receipt to Meta (required — Meta expects a fast response) |
| **Extract Message** | Parses Meta's nested JSON payload to pull out sender number, message text, and phone number ID |
| **Basic LLM Chain** | Sends the message text to the AI model with a system prompt, and receives the reply |
| **Groq Chat Model** | The actual AI model connection (Groq, running `openai/gpt-oss-120b`) |
| **Send WhatsApp Reply** | Posts the AI's reply back to the sender via Meta's Graph API |

---

## Common Issues

| Problem | Cause / Fix |
|---|---|
| Webhook verification fails in Meta dashboard | Verify token mismatch between n8n's "Check Verify Token" node and Meta's webhook config |
| No response on WhatsApp | Check n8n Executions tab for failed nodes; confirm workflow is **Active** |
| `401 Unauthorized` on Send WhatsApp Reply | Meta access token expired (temporary tokens last 24h) or missing `Bearer ` prefix in the credential |
| Groq node error | API key missing/invalid, or Groq account rate-limited |
| Webhook not reachable at all | ngrok tunnel closed/expired — restart ngrok and update the URL in Meta's webhook config |
| Message received but no reply generated | Check **Extract Message** output — confirm `text` field isn't empty (could be a non-text message type like image/audio, which this workflow doesn't handle yet) |

---

## Notes & Limitations

- This workflow only handles **text messages**. Images, voice notes, documents, etc. are ignored (the `Extract Message` code returns nothing if `message.text` isn't present).
- Meta's **temporary access token expires every 24 hours** during testing. For long-term/production use, generate a **permanent token** via a Meta System User.
- ngrok's free tier gives you a **new URL every restart** — you'll need to update Meta's webhook URL each time unless you're on a paid ngrok plan with a static domain, or move to a real server with a fixed domain.
- The AI has no memory between messages in this version — each message is treated independently. Adding conversation memory would require an additional node (e.g. a Memory/Buffer node keyed by sender number).
