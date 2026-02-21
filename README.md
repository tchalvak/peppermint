# peppermint
A job search an application agent

# 🌿 Peppermint

**Autonomous daily job search agent.** Finds, scores, and texts you one great job match every morning. Reply YES to apply, SKIP to pass.

-----

## How It Works

Every day at 8am, Peppermint:

1. **Ingests** jobs from three sources — your Indeed digest email, the Adzuna API, and direct web scraping
1. **Scores** each job against your criteria using Claude (the LLM) on a 0–10 rubric
1. **Selects** the top match above your threshold (default: 7/10)
1. **Texts** you a 2-sentence summary with score and highlights
1. **Waits** for your reply: `YES`, `SKIP`, or `INFO`
1. **Acts** — applies via Easy Apply or sends you the link, then logs everything

-----

## Setup

### 1. Clone and configure

```bash
git clone https://github.com/YOUR_USERNAME/peppermint.git
cd peppermint
cp .env.example .env
```

Edit `.env` with your API keys (see below). Edit `criteria.yaml` to set your job preferences.

### 2. Add GitHub Secrets

Go to your repo → Settings → Secrets and Variables → Actions. Add:

|Secret                 |Where to get it                                        |
|-----------------------|-------------------------------------------------------|
|`ANTHROPIC_API_KEY`    |console.anthropic.com                                  |
|`TWILIO_ACCOUNT_SID`   |twilio.com/console                                     |
|`TWILIO_AUTH_TOKEN`    |twilio.com/console                                     |
|`TWILIO_FROM_NUMBER`   |Your Twilio phone number                               |
|`TWILIO_TO_NUMBER`     |Your personal phone number                             |
|`INDEED_EMAIL_ADDRESS` |Your email that receives Indeed digests                |
|`INDEED_EMAIL_PASSWORD`|App password (Gmail: myaccount.google.com/apppasswords)|
|`ADZUNA_APP_ID`        |developer.adzuna.com (free tier)                       |
|`ADZUNA_APP_KEY`       |developer.adzuna.com (free tier)                       |

### 3. Set up the SMS webhook (for YES/SKIP replies)

Twilio needs a public URL to send your replies to. Options:

- **Free**: Use [Railway](https://railway.app) or [Render](https://render.com) to host `webhook.py`
- **Simple**: Use [ngrok](https://ngrok.com) for local testing

Point your Twilio number’s “A Message Comes In” webhook to: `https://YOUR_URL/sms`

### 4. Test manually

```bash
pip install -r requirements.txt
playwright install chromium
python main.py
```

Or trigger the GitHub Action manually from the Actions tab.

-----

## Weekly Tuning

Edit `criteria.yaml` to adjust your search. Key fields to tweak weekly:

```yaml
software:
  required_technologies:    # Add/remove hard-required tech
  preferred_industries:     # Shift industry focus
  min_salary_usd: 80000     # Raise or lower floor

local:
  max_walk_miles: 2.0       # Widen/narrow local radius
  scoring_weights:
    people_facing: 3        # Increase to prioritize social roles

global:
  min_score_to_notify: 7    # Lower to 6 if too few matches
```

Commit your changes and the next morning’s run picks them up automatically.

-----

## File Structure

```
peppermint/
├── criteria.yaml              ← Edit this weekly
├── main.py                    ← Daily orchestrator
├── webhook.py                 ← Handles YES/SKIP SMS replies
├── requirements.txt
├── agents/
│   ├── ingest.py              ← Job sourcing (email, API, scraper)
│   ├── score.py               ← LLM scoring
│   ├── notify.py              ← Twilio SMS
│   └── apply.py               ← Application handling
├── prompts/
│   ├── scoring_prompt.py      ← Core scoring rubric prompt
│   ├── sms_templates.py       ← SMS message templates
│   └── cover_letter_prompt.py ← Cover letter generation
├── logs/
│   └── applications.json      ← Auto-updated by bot
└── .github/workflows/
    └── daily.yml              ← Runs at 8am ET daily
```

-----

## Costs (estimated monthly)

|Service                                     |Cost                       |
|--------------------------------------------|---------------------------|
|Anthropic API (Claude Haiku, ~30 scores/day)|~$2–5/mo                   |
|Twilio SMS                                  |~$1/mo                     |
|Adzuna API                                  |Free tier                  |
|GitHub Actions                              |Free (2000 min/mo included)|
|**Total**                                   |**~$3–6/mo**               |

-----

## SMS Commands

|Reply |Action                                          |
|------|------------------------------------------------|
|`YES` |Attempt Easy Apply; fallback to opening job link|
|`SKIP`|Log as skipped, move on                         |
|`INFO`|Get full job details + direct link via SMS      |
|`STOP`|Pause Peppermint (standard Twilio opt-out)      |

-----

## Local Role Scoring

Local jobs near Wayland, NY are scored on a lifestyle rubric, not just salary:

- **+3** People-facing / customer contact
- **+2** Community impact (library, nonprofit, education, public service)
- **+1** Light physical activity (on your feet)
- **+1** Moderate physical activity (physical tasks)
- **+1** Salary ≥ $60k or reasonable for role type

A desk job at a local company will still appear but score lower than a hardware store associate or library aide.
