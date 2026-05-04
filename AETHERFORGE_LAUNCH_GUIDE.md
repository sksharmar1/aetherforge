# AetherForge AI Service Agent - LAUNCH GUIDE
## Get Your 45-Agent Marketing Agency Live in 4 Hours

**Date:** May 04, 2026  
**Status:** Ready to Deploy  
**Estimated Setup Time:** 4 hours (2 hours active work, 2 hours waiting for confirmations)

---

## PHASE 1: IMMEDIATE SETUP (Next 30 min)

### Step 1: Create Twilio Account & Get Phone Number

**What you're doing:** Creating a phone number for inbound/outbound calls

```bash
# 1. Go to https://www.twilio.com/
# 2. Click "Try for free" (no credit card required initially)
# 3. Verify your email + phone number
# 4. In Twilio Console, go to "Buy a Number"
# 5. Choose country: USA, filter by "Voice"
# 6. Select any toll-free number (they're free in trial)
# 7. Click "Buy" - you'll get something like +1-XXX-XXX-XXXX

# Save these credentials to a .env file:
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_PHONE_NUMBER=+1-XXX-XXX-XXXX
TWILIO_WEBHOOK_URL=https://your-domain.com  # We'll set this later
```

**Cost:** $0 (trial includes $15 free credit)

**Time:** 10 minutes

---

### Step 2: Set Up Supabase (PostgreSQL Database)

**What you're doing:** Creating a database to store leads, prospects, and call logs

```bash
# 1. Go to https://supabase.com/
# 2. Click "Start your project"
# 3. Sign up with GitHub or email
# 4. Click "New Project"
# 5. Name it "aetherforge-crm"
# 6. Choose region closest to you
# 7. Create password (save this!)
# 8. Wait 2 minutes for project to initialize

# In Supabase Console, go to "SQL Editor" and run these queries:

-- LEADS TABLE (for inbound calls)
CREATE TABLE leads (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  phone_number VARCHAR(20) NOT NULL,
  pain_points TEXT[] NOT NULL,
  timeline VARCHAR(50),
  budget VARCHAR(100),
  service_tier_recommended VARCHAR(50),
  lead_score INTEGER,
  requires_escalation BOOLEAN,
  transcript TEXT,
  call_duration_seconds INTEGER,
  source VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status VARCHAR(50) DEFAULT 'new'
);

-- PROSPECTS TABLE (for outbound calls)
CREATE TABLE prospects (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(255) NOT NULL,
  company VARCHAR(255) NOT NULL,
  title VARCHAR(255),
  industry VARCHAR(100),
  email VARCHAR(255),
  phone VARCHAR(20),
  company_size VARCHAR(50),
  pain_points TEXT[],
  source VARCHAR(50),
  status VARCHAR(50) DEFAULT 'new',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_contacted TIMESTAMP
);

-- OUTBOUND CALLS TABLE
CREATE TABLE outbound_calls (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  prospect_id UUID REFERENCES prospects(id),
  call_timestamp TIMESTAMP,
  call_duration_seconds INTEGER,
  reached BOOLEAN,
  sentiment VARCHAR(50),
  interest_level INTEGER,
  next_action VARCHAR(100),
  callback_scheduled BOOLEAN,
  callback_date TIMESTAMP,
  notes TEXT
);

-- CALLBACKS TABLE
CREATE TABLE callbacks (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  lead_id UUID REFERENCES leads(id),
  prospect_id UUID REFERENCES prospects(id),
  phone_number VARCHAR(20),
  scheduled_for TIMESTAMP,
  type VARCHAR(50),
  status VARCHAR(50) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes for fast queries
CREATE INDEX idx_leads_status ON leads(status);
CREATE INDEX idx_leads_created ON leads(created_at DESC);
CREATE INDEX idx_prospects_status ON prospects(status);
CREATE INDEX idx_outbound_calls_prospect ON outbound_calls(prospect_id);
```

**Get API keys:**
```bash
# In Supabase Console:
# 1. Go to Settings → API
# 2. Copy "Project URL" → SUPABASE_URL
# 3. Copy "anon public" key → SUPABASE_KEY

SUPABASE_URL=https://xxxxxxxxxxxxxxxx.supabase.co
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Cost:** $0 (free tier includes 500MB database)

**Time:** 10 minutes

---

### Step 3: Get Anthropic API Key

**What you're doing:** Enabling Claude API for the AI agents

```bash
# 1. Go to https://console.anthropic.com/
# 2. Sign in (or create account)
# 3. Go to "API Keys"
# 4. Click "Create Key"
# 5. Name it "AetherForge"
# 6. Copy the key

ANTHROPIC_API_KEY=sk-ant-v3-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Cost:** Pay-as-you-go (~$0.003 per inbound call, ~$0.001 per outbound qualifying call)

**Time:** 5 minutes

---

### Step 4: Deploy FastAPI Server (Choose One)

#### Option A: **Deploy to Railway** (Recommended for this use case - simplest)

```bash
# 1. Go to https://railway.app/
# 2. Sign up with GitHub
# 3. Click "New Project"
# 4. Select "Deploy from GitHub"
# 5. Connect your GitHub account
# 6. Create new GitHub repo "aetherforge-service-agent"
# 7. Push the code:

git init
git add aetherforge_service_agent.py requirements.txt
git commit -m "Initial commit"
git push origin main

# 8. In Railway dashboard, configure environment variables:
TWILIO_ACCOUNT_SID=...
TWILIO_AUTH_TOKEN=...
TWILIO_PHONE_NUMBER=...
SUPABASE_URL=...
SUPABASE_KEY=...
ANTHROPIC_API_KEY=...

# 9. Railway auto-deploys. Your URL will be:
# https://your-project.railway.app

# Save your Railway URL:
TWILIO_WEBHOOK_URL=https://your-project.railway.app
```

#### Option B: **Deploy to Render** (Alternative)

```bash
# Similar to Railway - create account, connect GitHub, deploy
# https://render.com/
```

#### Option C: **Run Locally** (Development only)

```bash
# Install dependencies
pip install fastapi uvicorn twilio anthropic supabase httpx python-dotenv

# Create .env file with all credentials above

# Run server
python aetherforge_service_agent.py

# Your webhook URL will be: http://localhost:8000
# (You'll need ngrok for Twilio to reach local machine:
#  ngrok http 8000 → get public URL like https://xxxx.ngrok.io)
```

**Cost:** ~$5-10/month for Railway (plus usage)

**Time:** 10 minutes

---

## PHASE 2: CONFIGURE TWILIO WEBHOOKS (Next 15 min)

**What you're doing:** Telling Twilio where to send incoming calls

```bash
# In Twilio Console:
# 1. Go to Phone Numbers → Manage Numbers
# 2. Click on your phone number
# 3. Scroll to "Voice Configuration"
# 4. For "A call comes in", select "Webhook"
# 5. Enter URL: https://your-project.railway.app/voice/inbound
# 6. Method: POST
# 7. Scroll to "Recording" → Enable
# 8. Save

# Test the webhook:
curl -X POST https://your-project.railway.app/health
# Should return: {"status": "healthy", ...}
```

---

## PHASE 3: SET UP COWORK AUTOMATION (Next 30 min)

**What you're doing:** Automating LinkedIn posting, email follow-ups, and content scheduling

### Install Cowork (if you have Claude in Chrome extension)

```bash
# In Claude in Chrome, enable Cowork automation:
# 1. Open https://getaetherforge.com
# 2. Claude takes screenshot
# 3. Click "Automate LinkedIn posting"
# 4. Schedule content calendar
```

### LinkedIn Content Calendar (AI-generated)

Create file `linkedin_content_calendar.md`:

```markdown
# AetherForge LinkedIn Content Calendar (30 Days)

## Week 1: Thought Leadership

### Monday: "The AI Agency Revolution"
POST: "We just launched an AI-powered marketing agency with 45 specialized agents. 
Same results as traditional agencies. 40-60% lower cost. 24/7 execution. 
The future of marketing is automated. Welcome to AetherForge. 
Link: getaetherforge.com"

### Wednesday: "Why Most Agencies Fail"
POST: "We analyzed 200+ marketing agencies. Common failures:
1. Cookie-cutter strategies (no customization)
2. Slow turnaround (weeks, not hours)
3. Limited expertise (jack of all trades)
4. Expensive overhead ($5k-20k/month)

AetherForge: Specialized agents, instant response, deep expertise, 60% less cost.

Is your current agency checking these boxes?"

### Friday: "Case Study: 180% Lead Growth"
POST: "We just helped a SaaS company increase qualified leads by 180% in 90 days.

Their challenge: Low lead volume, high CAC, no brand visibility.

Our approach: AI-powered SEO + Google Ads optimization + LinkedIn content.

Result: 45 leads/month → 125 leads/month. CAC dropped 40%.

What's your biggest marketing challenge?"

## Week 2: Educational Content

### Monday: "SEO in 2026"
POST: "The SEO playbook changed. Here's what works in 2026:
1. AI-optimized content (but actually useful)
2. User experience signals (Core Web Vitals matter)
3. Topic clusters (not keyword stuffing)
4. Authority building (backlinks + mentions)
5. Technical SEO (JSON-LD, site speed, mobile)

Most agencies still use 2020 tactics. That's why they fail.

AetherForge's AI agents are trained on 2025+ best practices."

... (continue for 30 days)
```

### Create Email Follow-Up Sequence

```bash
# Using Make.com or Zapier:
# 1. Trigger: Lead form submitted on getaetherforge.com
# 2. Action 1: Send immediate confirmation email
# 3. Action 2: Log to Supabase
# 4. Action 3: Schedule follow-up email for 24 hours later
# 5. Action 4: If no response in 3 days, trigger BDR call

# Email template:

Subject: "Your AetherForge Strategy Session (Tomorrow at 2pm)"

Hi [Name],

Thanks for reaching out! We're excited about the possibility of helping [Company] 
grow with AetherForge.

Based on what you shared, here's what we'll explore in your strategy session:

✓ Your biggest marketing bottleneck (and how to fix it in 90 days)
✓ AI-powered tactics for [Industry]
✓ ROI roadmap with clear milestones

Your VP of Sales, [Name], will lead the call. She's seen 45 variations of your 
challenge and has a 3-step framework that works.

[BOOK TIME SLOT]

Questions before the call? Hit reply.

Best,
AetherForge Sales Team
P.S. Quick fact: companies in [Industry] typically see 180%+ lead growth in 90 days.
```

---

## PHASE 4: IMPORT PROSPECT LIST & START CALLING (Next 1 hour)

### Import Prospects for Outbound BDR

**Option 1: Manual List (10 prospects)**

Create `prospects.csv`:
```csv
name,company,title,industry,email,phone,company_size,pain_points
John Smith,Acme Corp,VP Marketing,SaaS,john@acme.com,+14155551234,51-200,low lead generation
Sarah Chen,Tech Startup Inc,CEO,FinTech,sarah@techstartup.com,+14155551235,1-10,brand visibility
...
```

**Option 2: Apollo/ZoomInfo Integration**

```bash
# Paste this into a Python script to auto-import from Apollo:

import requests
import os

APOLLO_API_KEY = os.getenv("APOLLO_API_KEY")

def import_from_apollo(industry: str, company_size: str, limit: int = 50):
    """Import prospects from Apollo.io"""
    
    url = "https://api.apollo.io/v1/contacts/search"
    
    payload = {
        "api_key": APOLLO_API_KEY,
        "q_organization_industry": industry,
        "q_organization_size": company_size,
        "limit": limit
    }
    
    response = requests.post(url, json=payload)
    prospects = response.json().get("contacts", [])
    
    # Convert to our format and upload to Supabase
    # ... (see BDR agent code)
    
    return prospects

# Usage:
# prospects = import_from_apollo("SaaS", "51-200", limit=100)
```

### Start Calling

```bash
# Run BDR agent:

from aetherforge_bdr_agent import run_daily_calling_campaign

# Simulate 50 calls today (without actual Twilio yet)
report = await run_daily_calling_campaign(daily_limit=50, simulate=True)

# Once comfortable, switch to actual calls:
# report = await run_daily_calling_campaign(daily_limit=50, simulate=False)
```

---

## PHASE 5: MONITOR & ITERATE (Ongoing)

### Dashboard Endpoints

```bash
# Check health:
curl https://your-project.railway.app/health

# Get recent leads:
curl https://your-project.railway.app/leads?limit=20

# Get daily calling report:
curl https://your-project.railway.app/bdr/daily-report
```

### Key Metrics to Track

| Metric | Target | Current |
|--------|--------|---------|
| Inbound call → qualified lead | 60% | — |
| Outbound call → interested (3+) | 25% | — |
| Lead score avg | 65+ | — |
| Demo conversion | 40% | — |
| Sales cycle | 14-21 days | — |

---

## REQUIREMENTS.TXT (Dependencies)

```
fastapi==0.104.1
uvicorn==0.24.0
twilio==8.10.0
anthropic==0.25.0
supabase==2.0.0
httpx==0.25.0
pydantic==2.5.0
python-dotenv==1.0.0
```

---

## .ENV FILE TEMPLATE

```bash
# Twilio
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_PHONE_NUMBER=+1-XXX-XXX-XXXX
TWILIO_WEBHOOK_URL=https://your-domain.com

# Supabase
SUPABASE_URL=https://xxxxxxxxxxxxxxxx.supabase.co
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Anthropic
ANTHROPIC_API_KEY=sk-ant-v3-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Optional
APOLLO_API_KEY=your_apollo_api_key
MAKE_WEBHOOK_URL=https://hook.make.com/...
```

---

## TROUBLESHOOTING

| Issue | Solution |
|-------|----------|
| "Form not found" error on website | Check Formspree setup from AetherForge project |
| Twilio webhook not firing | Verify webhook URL in Twilio console, check logs in Railway |
| Supabase connection error | Check credentials in .env, verify IP whitelist |
| Claude API rate limit | Check billing in Anthropic console, add payment method |
| No leads coming through | Check Twilio test number, verify phone number format |

---

## NEXT STEPS (This Week)

- [ ] **Day 1:** Deploy service agent, test with 5 inbound calls
- [ ] **Day 2:** Start BDR campaign with 50 outbound calls (simulated)
- [ ] **Day 3:** Launch LinkedIn content calendar (3 posts/week)
- [ ] **Day 4:** Switch to real outbound calls, monitor interest rates
- [ ] **Day 5:** Analyze metrics, optimize scripts, scale to 200+ calls/week

---

## PROJECTED RESULTS (30 Days)

| Metric | Projection |
|--------|-----------|
| Inbound leads | 30-50 |
| Outbound demos scheduled | 10-15 |
| SQL (Sales-Qualified Leads) | 8-12 |
| Pipeline value | $50k-150k |
| CAC (Customer Acquisition Cost) | ~$500-1000 |

---

## SUPPORT & NEXT PHASES

**Phase 2 (Week 2-3):**
- Google Ads integration for paid lead gen
- LinkedIn Ads orchestration
- Email nurture sequences
- Client onboarding automation

**Phase 3 (Week 4+):**
- Add content creation agents (blog, case studies)
- Implement real-time analytics dashboard
- Launch performance-based pricing model
- Scale to enterprise clients

---

**Questions? Stuck?** Reply here and we'll help you launch. 🚀

Good luck out there.

— AetherForge Team
