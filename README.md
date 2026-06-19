# 🛡️ telegram-scam-detector-n8n

An AI-powered, multi-agent n8n workflow that automatically detects scams and fraudulent content shared via Telegram. When a user sends a URL or promotional message to a connected Telegram bot, the workflow runs five parallel AI investigations and delivers a comprehensive Arabic-language fraud report back to the user.

---

## 📋 Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Agent Architecture](#agent-architecture)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Setup & Installation](#setup--installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Output Format](#output-format)
- [Scoring System](#scoring-system)
- [Edge Cases & Fallbacks](#edge-cases--fallbacks)
- [Repository Info](#repository-info)

---

## Overview

This workflow is designed to help Arabic-speaking users identify potential online scams before engaging with suspicious links or offers. It combines real-time web intelligence (via SerpAPI) with large language model reasoning (via OpenAI) across five specialized sub-agents, then synthesizes their findings into a single, structured fraud report delivered directly in Telegram.

**Key capabilities:**
- Analyzes URLs and/or raw promotional text
- Classifies the source platform (Facebook, Instagram, TikTok, etc.)
- Runs five parallel AI investigations simultaneously
- Produces a severity-scored Arabic fraud report (1–10 scale)
- Delivers the report back to the Telegram chat automatically

---

## How It Works

```
Telegram Message Received
        │
        ▼
[URL Extraction & Platform Classification]
        │
        ├──────────────────────────────────────────────────┐
        │                                                  │
        ▼                                                  ▼
  [URL Detected]                                   [Text Only]
        │                                                  │
        ├─── Agent 1: Domain & Technical Details           │
        ├─── Agent 2: Search Engine Signals                │
        ├─── Agent 3: Product & Pricing Patterns           │
        ├─── Agent 4: Content Analysis                     │
        └─── Agent 5: Text Analysis ◄─────────────────────┘
                │
                ▼
         [Merge & Aggregate]
                │
                ▼
      [Evaluator Agent — Final Verdict]
                │
                ▼
    [Telegram Report Sent to User]
```

1. **Trigger:** A message is received by the Telegram bot.
2. **Parsing:** A JavaScript node extracts the URL (if present) and classifies the source platform.
3. **Parallel Analysis:** Five AI agents run simultaneously, each investigating a different dimension of the input.
4. **Synthesis:** All five outputs are merged, aggregated, and passed to the Evaluator Agent.
5. **Report:** The Evaluator computes a final scam score and sends a formatted Arabic report back to Telegram.

---

## Agent Architecture

| Agent | Input | Role |
|---|---|---|
| **Domain & Technical Details** | URL | Checks domain age (< 6 months = red flag), SSL/HTTPS status, and high-risk TLDs (`.xyz`, `.top`, `.click`, etc.) |
| **Search Engine Signals** | URL | Searches for scam reports on Reddit, Trustpilot, ScamAdviser, SiteJabber, and checks overall search reputation |
| **Product & Pricing Patterns** | URL | Detects exaggerated discounts, luxury goods at suspiciously low prices, and fake endorsements |
| **Content Analysis** | URL | Evaluates website content for missing company info, fake addresses, and deceptive design patterns |
| **Text Analysis** | Raw Message Text | Analyzes the promotional message for logical contradictions, grammatical errors, artificial urgency, and manipulative tactics |
| **Evaluator** | All 5 outputs | Synthesizes findings, computes a weighted scam score (1–10), and generates the final Arabic Telegram report |

---

## Tech Stack

| Component | Technology |
|---|---|
| Workflow Engine | [n8n](https://n8n.io/) |
| Trigger | Telegram Bot (via n8n Telegram Trigger node) |
| AI Models | OpenAI `gpt-4.1-mini` (Evaluator), `gpt-4o-mini` (Sub-agents) |
| Search Intelligence | [SerpAPI](https://serpapi.com/) — Google Search & Shopping |
| Output Channel | Telegram Bot (Send Message node) |
| Language | JavaScript (URL parsing node) |
| Report Language | Arabic |

---

## Prerequisites

Before importing and running this workflow, you will need:

- **n8n instance** (self-hosted or cloud) running version 1.0+
- **Telegram Bot Token** — Create a bot via [@BotFather](https://t.me/BotFather)
- **OpenAI API Key** — [platform.openai.com](https://platform.openai.com/)
- **SerpAPI Key** — [serpapi.com](https://serpapi.com/)

---

## Setup & Installation

### 1. Import the Workflow

1. Open your n8n instance.
2. Go to **Workflows → Import from File**.
3. Upload `Scam_detection.json`.

### 2. Configure Credentials

You need to set up three credentials in n8n:

**Telegram:**
1. Go to **Settings → Credentials → New**.
2. Select **Telegram API**.
3. Enter your bot token from BotFather.

**OpenAI:**
1. Go to **Settings → Credentials → New**.
2. Select **OpenAI API**.
3. Enter your OpenAI API key.

**SerpAPI:**
1. Go to **Settings → Credentials → New**.
2. Select **SerpAPI**.
3. Enter your SerpAPI key.

### 3. Link Credentials to Nodes

After import, open each node and assign the correct credentials:
- **Telegram Trigger** and **Send a text message** → your Telegram credential
- All **OpenAI Chat Model** nodes → your OpenAI credential
- All **SerpAPI** nodes → your SerpAPI credential

### 4. Update the Telegram Chat ID

In the **Send a text message** node, update the `chatId` field to the Telegram chat or user ID where reports should be delivered.

### 5. Activate the Workflow

Toggle the workflow to **Active** in n8n. Your Telegram bot will now respond to messages.

---

## Configuration

### Models

| Node | Default Model | Notes |
|---|---|---|
| Evaluator (final agent) | `gpt-4.1-mini` | Can be upgraded to `gpt-4o` for higher accuracy |
| All sub-agents (×5) | `gpt-4o-mini` | Cost-efficient; suitable for structured analysis tasks |

To change a model, open the corresponding **OpenAI Chat Model** node and select a different model from the dropdown.

### Supported Platforms (Auto-Detected)

The JavaScript parser automatically identifies URLs from: Facebook, Instagram, Snapchat, Twitter/X, TikTok, YouTube, and LinkedIn. Unrecognized URLs are classified as `unknown` and still proceed through the full analysis pipeline.

---

## Usage

Simply send a message to your Telegram bot containing:

- **A URL** — e.g., a link to a suspicious website or product page
- **A promotional message** — e.g., a forwarded offer, giveaway announcement, or job offer text
- **Both** — the workflow handles combined inputs

The bot will respond within seconds with a full fraud analysis report.

---

## Output Format

The final report is delivered in Arabic directly to Telegram, structured as follows:

```
🔍 تقرير الكشف عن الاحتيال
━━━━━━━━━━━━━━━━━━━━━━
🌐 تحليل الرابط (الأولوية القصوى)
🔒 التفاصيل التقنية والدومين:
[Domain age, SSL status, TLD analysis]

📡 إشارات محركات البحث:
[Scam reports, forum warnings, search reputation]

━━━━━━━━━━━━━━━━━━━━━━
📝 تحليل النص والمحتوى
🏷️ أنماط التسعير والمنتجات:
[Pricing anomalies, fake endorsements]

📄 تحليل المحتوى:
[Website content evaluation]

🧠 تحليل الرسالة الترويجية:
[Promotional text analysis]

━━━━━━━━━━━━━━━━━━━━━━
📊 نتيجة التقييم النهائي
🎯 درجة الخطورة: X / 10
🚦 مستوى التهديد: [Severity Level]

━━━━━━━━━━━━━━━━━━━━━━
✅ التوصية النهائية
[Action recommendation]

━━━━━━━━━━━━━━━━━━━━━━
⚠️ إخلاء المسؤولية: ...
```

---

## Scoring System

The Evaluator Agent computes a weighted scam score from **1 to 10** using a strict priority hierarchy:

### URL Health (Primary — sets the score floor)

| Signal | Condition | Impact |
|---|---|---|
| SSL/HTTPS | HTTP only or invalid SSL | Score cannot go below **8** |
| SSL/HTTPS | Unconfirmed | Score cannot go below **5** |
| Domain Age | Under 6 months | Score cannot go below **7** |
| Domain Age | Unconfirmed | Score cannot go below **4** |
| TLD | High-risk (`.xyz`, `.top`, `.click`, `.site`, etc.) | **+2** to floor |
| TLD | Uncommon but not high-risk | **+1** to floor |

### Search Engine Signals (Secondary)

| Signal | Impact |
|---|---|
| Explicit phishing/scam listing found | **+2** |
| Warnings on Reddit, Trustpilot, etc. | **+1** |
| No search presence at all | **+1** |
| Verified positive reviews | **−1** (never below URL floor) |

### Content & Text Signals (Tertiary)

| Signal | Impact |
|---|---|
| Logical contradiction in the offer | **+1** |
| Fake endorsements or government impersonation | **+1** |
| Artificial urgency or pressure language | **+1** |
| No company info or fake address | **+1** |

### Severity Levels

| Score | Level | Meaning |
|---|---|---|
| 1–2 | آمن ✅ | No significant red flags |
| 3–4 | منخفض ⚠️ | Minor concerns — proceed with caution |
| 5–6 | متوسط 🟡 | Multiple suspicious signals — verify before acting |
| 7–8 | عالٍ 🔴 | Strong fraud indicators — avoid engaging |
| 9–10 | حرج 🚨 | Almost certainly a scam — do not interact |

---

## Edge Cases & Fallbacks

| Scenario | Behavior |
|---|---|
| Any of the 5 agent outputs is missing | Returns a structured JSON error; no partial report sent |
| All agents return "unable to retrieve" | Default score of 5; report notes insufficient data |
| HTTP-only + high-risk TLD both confirmed | Minimum score forced to **9** |
| Both Domain and Search Engine agents flag scam | Minimum score forced to **8** |
| Text-only input (no URL) | URL-based agents return no data; Text Analysis still runs |

---

## Repository Info

**Recommended GitHub Repository Name:**
```
telegram-scam-detector-n8n
```

**Recommended Description:**
```
An n8n workflow that uses multi-agent AI (OpenAI + SerpAPI) to automatically analyze and score suspicious URLs and messages shared via Telegram, delivering Arabic-language fraud detection reports in real time.
```

**Suggested GitHub Topics/Tags:**
`n8n` `automation` `scam-detection` `fraud-detection` `telegram-bot` `openai` `serpapi` `arabic` `ai-agents` `langchain`

---

## Disclaimer

Reports generated by this workflow are for informational purposes only. A low risk score does not guarantee a website is safe, and a high score does not definitively confirm fraud. Always use personal judgment and do not rely solely on automated analysis.
