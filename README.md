# Mittu — WhatsApp AI Agent for Indian Small Businesses

> A production-grade agentic AI system that helps kirana stores, medical shops, dairy vendors and other small businesses manage orders, track sales, and monitor udhaar — all through natural WhatsApp conversation in Hindi, English, or Gujarati.

![Python](https://img.shields.io/badge/Python-3.11+-blue)
![Flask](https://img.shields.io/badge/Flask-3.x-green)
![Groq](https://img.shields.io/badge/AI-Groq%20LLaMA%203.3-orange)
![Supabase](https://img.shields.io/badge/DB-Supabase-darkgreen)
![Status](https://img.shields.io/badge/Status-Live%20%F0%9F%9F%A2-brightgreen)

---

## What Mittu Does

Shop owners send natural WhatsApp messages — Mittu understands and acts:

| Owner says | Mittu does |
|---|---|
| "Suresh ne 2kg aata liya" | Saves order, confirms with details |
| "Nia ka order — milk Rs 30, waffers Rs 45" | Auto-calculates total Rs 75, confirms |
| "Aaj ka report dikhao" | Shows order count and revenue |
| "Ramesh ko Rs 200 ka udhaar diya" | Saves to digital bahi khata |
| "Ramesh ka kitna udhaar hai?" | Shows total pending amount |

No app download. No training. Just WhatsApp.

---

## Architecture — Multi-Agent Orchestration

```
WhatsApp Message
      ↓
  Twilio Webhook
      ↓
  Flask Server (Render)
      ↓
  ┌─────────────────────────────┐
  │      ORCHESTRATOR           │
  │  Detects intent via LLM     │
  │  Routes to correct agent    │
  └─────────────────────────────┘
      ↓              ↓            ↓           ↓          ↓
  ORDER          REPORT        UDHAAR      UPDATE      GREETING
  Agent          Agent         Agent       Agent       Handler
      ↓              ↓            ↓
  Supabase      Supabase      Supabase
  orders        orders        udhaar
  table         aggregate     table
      ↓
  Groq LLaMA 3.3
  (reply generation)
      ↓
  WhatsApp Reply
```

Each agent has a **single responsibility**. The orchestrator never does the work itself — it only routes.

---

## Key Technical Features

### Conversation Memory
Last 5 messages stored per shop in Supabase. Enables context resolution:
- Owner says "uska price Rs 50 hai" → Mittu links it to the previous order

### Auto-Calculation
Multi-item orders with individual prices are summed automatically:
- "daal Rs 100 aur dudh Rs 50" → confirmed as total Rs 150

### Plan-Gated Features
Single `PLAN_LIMITS` config controls all tier restrictions:

```python
PLAN_LIMITS = {
    "free":    { "orders_per_day": 10, "udhaar_persons": 0, ... },
    "plan99":  { "orders_per_day": 999999, "udhaar_persons": 5, ... },
    "plan199": { "orders_per_day": 999999, "udhaar_persons": 999999, ... },
}
```

Changing a limit = changing one number. No code rewrite.

### Language Detection + Plan Gating
- Rule-based for Devanagari and Gujarati script (instant, no AI cost)
- Groq-based for Roman script (context-aware, avoids false Hinglish)
- Gujarati and Marathi gated behind Rs 199 plan

### Hard-Coded Fallbacks
Server never sends empty message even if AI fails:
```python
fallbacks = {
    "HINDI":    "Kuch technical gadbad hua. Thodi der baad try karein. - Mittu",
    "GUJARATI": "Thodi technical samasya. Pachhi try karo. - Mittu",
    "ENGLISH":  "Something went wrong. Please try again. - Mittu",
}
```

---

## Tech Stack

| Layer | Technology | Cost |
|---|---|---|
| WhatsApp Gateway | Twilio / Meta WhatsApp Cloud API | Free tier |
| Backend | Python + Flask | Free |
| Hosting | Render | Free tier |
| AI / LLM | Groq — LLaMA 3.3 70B | Free tier |
| Database | Supabase (PostgreSQL) | Free tier |
| Uptime | UptimeRobot | Free |
| Dashboard | GitHub Pages | Free |

**Total infrastructure cost: Rs 0/month** for early stage

---

## Business Model

| Plan | Price | Key Features |
|---|---|---|
| Shuruat | Rs 0/month | 10 orders/day, Hindi + English |
| Chota Boss | Rs 99/month | Unlimited orders, revenue reports, udhaar (5 customers) |
| Asli Boss | Rs 199/month | Everything + Gujarati/Marathi, unlimited udhaar, stock management |

Target market: 12M+ small businesses in India currently using notebooks and WhatsApp manually.

---

## Database Schema

```sql
shops         — phone, name, shop_type, owner_name, plan, language
orders        — shop_id, customer_name, items, amount, status
udhaar        — shop_id, customer_name, amount, description, status
conversations — shop_id, role, message (last 10 per shop)
reminders     — shop_id, reminder_type, message, remind_at, status
```

---

## Onboarding Flow

New shop owners are guided through a 4-step setup:

```
Step 0 → Welcome message, ask shop name
Step 1 → Save shop name, ask business type
Step 2 → Save business type, ask owner name  
Step 3 → Save owner name, complete onboarding
```

After onboarding, Mittu knows the shop context and gives business-type-relevant examples and responses.

---

## Running Locally

```bash
git clone https://github.com/jinaljoshi92/mittu
cd mittu
pip install -r requirements.txt

# Create .env file
cp .env.example .env
# Fill in your credentials

python main.py
```

### Environment Variables

```
TWILIO_SID=
TWILIO_TOKEN=
TWILIO_NUMBER=
SUPABASE_URL=
SUPABASE_KEY=
GROQ_KEY=
```

---

## Project Status

- ✅ Core agents — Order, Report, Udhaar, Update, Greeting
- ✅ Multi-language support — Hindi, English, Hinglish, Gujarati, Marathi
- ✅ Conversation memory and context resolution
- ✅ Auto-calculation for multi-item orders
- ✅ Plan-gated features with subscription tiers
- ✅ Live dashboard with real-time Supabase data
- 🔄 Meta WhatsApp Business API integration (in progress)
- 🔄 Reminder agent — scheduled udhaar and restock reminders
- 🔄 Sarvam AI integration for improved Indic language quality

---

## Why This Project

India has 12 million+ small shop owners who manage everything in physical notebooks and informal WhatsApp messages. Existing solutions (Wati, Interakt) target mid-size businesses at Rs 2,000-5,000/month. Mittu targets the underserved bottom of the market — the vegetable vendor, the dairy shop, the neighbourhood kirana — at Rs 0 to Rs 199/month, in their own language, with zero technology learning curve.

**Core differentiators:**
- Only WhatsApp — no app download ever
- Hindi, Gujarati, Marathi — India-first languages
- Built for businesses making Rs 5,000-50,000/month, not D2C brands
- Rs 0 to start — lowest barrier to entry in the market

---

*Built by Jinal H Joshi — [LinkedIn](https://linkedin.com/in/jinalhjoshi) · [GitHub](https://github.com/jinaljoshi92)*
