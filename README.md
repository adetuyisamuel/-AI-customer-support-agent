# AI Customer Support Agent — n8n + GPT-4o

An intelligent customer support agent built in n8n. Incoming support messages are answered automatically by GPT-4o using a custom knowledge base. Complex queries are escalated to a human agent with full context attached — so nothing falls through the gaps and no customer waits longer than necessary.

---

## The Problem

Most small businesses handle support manually — every question answered one by one, often hours after the customer asked. Repetitive questions about pricing, refunds, and shipping consume hours of team time every week. And when the team is busy, customers wait.

---

## The Solution

Every inbound support message is processed instantly by GPT-4o. The AI answers from a trained knowledge base, categorises the ticket, and logs it automatically. If the question is outside the knowledge base or too complex, it escalates to a human with a full context summary — and the customer gets an acknowledgement immediately so they know someone is on it.

---

## What Happens Automatically

```
Customer sends message (website chat, email, or form)
        ↓
Parse name, email, message, channel
        ↓
GPT-4o checks knowledge base
Returns: answer OR escalation flag
        ↓
IF escalate = false             IF escalate = true
        ↓                               ↓
Send AI answer by email         Log ticket — Pending Human Review
Log ticket — Resolved by AI     Slack alert to support team
                                Send acknowledgement email to customer
```

---

## Results

- 80% of common queries resolved automatically with zero human involvement
- Response time reduced from hours to under 10 seconds
- Human agents only handle complex or sensitive issues
- Every ticket logged with category, answer, and escalation reason
- No customer goes unacknowledged — even escalations get an instant reply

---

## Tools and Stack

| Tool | Purpose |
|---|---|
| n8n | Workflow automation engine |
| GPT-4o (OpenAI) | Knowledge base Q&A and escalation decisions |
| Gmail | Send AI answers and escalation acknowledgements |
| Google Sheets | Full ticket log — resolved and escalated |
| Slack | Real-time escalation alerts to support team |

---

## Workflow Nodes

1. **Webhook** — Receives inbound support message
2. **Set Node** — Maps customer name, email, message, channel
3. **OpenAI GPT-4o** — Answers from knowledge base or returns escalation JSON
4. **Code Node** — Parses and validates GPT response safely
5. **IF Node** — Routes escalation vs resolution
6. **Gmail — AI Answer** — Sends personalised answer email to customer
7. **Sheets — Resolved** — Logs ticket with category and AI answer
8. **Sheets — Escalated** — Logs ticket with escalation reason
9. **Slack — Alert** — Posts full context to #support-escalations channel
10. **Gmail — Acknowledgement** — Sends holding email to customer for escalated tickets
11. **Respond Node** — Returns webhook confirmation

---

## The GPT System Prompt Structure

The AI answers only from the knowledge base you provide. The prompt forces structured JSON output every time:

**Resolved response:**
```json
{
  "escalate": false,
  "answer": "your answer here",
  "category": "billing|technical|general|returns|shipping"
}
```

**Escalation response:**
```json
{
  "escalate": true,
  "reason": "one sentence reason"
}
```

Temperature is set to 0.3 — low enough for consistency, with slight flexibility for natural language responses.

---

## Knowledge Base

The knowledge base lives inside the GPT system prompt. To customise it for any business, replace the content between the `---` markers with that business's actual information:

```
Business hours: Monday to Friday, 9am to 5pm
Refund policy: Full refund within 30 days
Pricing: Starter $49/month, Pro $99/month
Common issues: login, billing, cancellation steps
```

Any question outside the knowledge base automatically escalates to a human — the AI never makes up answers.

---

## Google Sheets Structure

Create a sheet called `Support Tickets` with these columns:

| Column | Description |
|---|---|
| Timestamp | When the message was received |
| Customer Name | From the inbound message |
| Email | Customer email address |
| Message | Original customer message |
| Channel | website, email, chat, WhatsApp |
| Category | billing, technical, general, returns, shipping |
| AI Answer | The answer GPT gave (blank if escalated) |
| Escalated | Yes or No |
| Escalation Reason | Why it was escalated (blank if resolved) |
| Status | Resolved by AI or Pending Human Review |

---

## How to Import and Use

**Step 1 — Import the workflow**
- Open n8n → Workflows → Import from file
- Upload `ai_customer_support_workflow.json`

**Step 2 — Connect credentials**
- OpenAI API key
- Gmail OAuth
- Google Sheets OAuth
- Slack OAuth

**Step 3 — Update these values**
- Replace `YOUR_GOOGLE_SHEET_ID` in both Sheets nodes
- Replace `#support-escalations` with your Slack channel
- Replace `[Your Business Name]` in both email templates
- Update the knowledge base inside the GPT system prompt

**Step 4 — Connect to your chat widget or form**
- Copy the webhook URL from the Webhook node
- Paste it as the form action or chat widget endpoint
- Or use it with Typeform, Tally, or any form tool

**Step 5 — Test**
Send a POST request with:
```json
{
  "name": "James Adeyemi",
  "email": "james@example.com",
  "message": "What is your refund policy?",
  "channel": "website"
}
```

Expected result: AI answers with the refund policy, logs the ticket as resolved, no escalation.

Then test escalation:
```json
{
  "name": "Sarah Osei",
  "email": "sarah@example.com",
  "message": "I was charged twice last month and my account shows a different amount than my invoice. I need this resolved urgently.",
  "channel": "website"
}
```

Expected result: Escalated to human, Slack alert fired, acknowledgement email sent.

---

## Customisation

- **Add WhatsApp support** — replace the webhook trigger with a WhatsApp Business API trigger
- **Add conversation history** — pass previous messages into the GPT prompt for multi-turn conversations
- **Add sentiment detection** — add a second GPT call to detect angry or urgent messages and prioritise them
- **Swap Gmail for SendGrid** — replace Gmail nodes with HTTP Request nodes to SendGrid API
- **Add ticket IDs** — generate a unique ticket ID in a Code node and include it in all emails

---

## About This Project

Built as part of a professional automation portfolio. Demonstrates AI-powered conditional logic — GPT reads intent, routes appropriately, and ensures every customer gets an immediate response whether the AI can help or not.

**Tools demonstrated:** n8n · GPT-4o · Gmail · Google Sheets · Slack · IF logic · Code node · JSON parsing · Error escalation
