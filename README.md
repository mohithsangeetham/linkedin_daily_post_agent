# LinkedIn Post Agent — AI / Tech / Data Science

A Google Colab notebook that generates a daily LinkedIn post draft + matching image, then emails both to you for review before posting. Built for data scientists, ML engineers, and tech professionals who want a consistent LinkedIn presence without spending hours writing.

---

## What it does

Every time you run it:

1. Fetches today's top AI/tech stories from Hacker News and recent papers from arXiv
2. Picks a topic from a rotating bank — no repeats for 7 days
3. Uses **Claude** (`claude-opus-4-7`) to write a LinkedIn post in your voice, grounded in real news
4. Feeds the post directly to **Gemini Nano Banana 2** (`gemini-3.1-flash-image-preview`) to generate a matching 1280×720 PNG image
5. Emails you the draft + image for review — you approve, tweak, and post manually

You stay in full control. Nothing goes to LinkedIn without you seeing it first.

---

## How the image generation works

The Claude-generated post text is passed **directly** into Gemini as the image prompt — no intermediate prompt engineering, no rewriting. Gemini reads the actual post content and generates an image that visually represents it.

This approach produces the tightest content-image alignment because:
- No information is lost in translation
- Gemini interprets the specific scenario, emotion, and objects described in the post
- Concrete, story-driven posts produce the most accurate images

The only instruction added is: *"Create a professional 16:9 landscape image. Do not include any text in the image."*

---

## Requirements

- **Google Colab Pro** (for Scheduled Notebooks and reliable runtime)
- **Anthropic API key** — get one at [console.anthropic.com](https://console.anthropic.com)
- **Google AI Studio API key** — get one at [aistudio.google.com](https://aistudio.google.com)
- **Gmail account** with 2FA enabled (for sending draft emails)
- **Gmail App Password** — 16-character password from [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)

---

## Setup

### Step 1 — Upload the notebook to Colab

Go to [colab.research.google.com](https://colab.research.google.com) → File → Upload notebook → select `linkedin_agent_v2.ipynb`.

### Step 2 — Add secrets

Click the 🔑 **Secrets** icon in the left sidebar. Add all four secrets below. Toggle **Notebook access ON (blue)** for each one.

| Secret Name | What to put in the Value field |
|---|---|
| `ANTHROPIC_API_KEY` | Your Claude API key (starts with `sk-ant-...`) |
| `GEMINI_API_KEY` | Your Google AI Studio API key |
| `GMAIL_ADDRESS` | Your Gmail address |
| `GMAIL_APP_PASSWORD` | Your 16-character Gmail App Password |

> Never paste secret values into the code cells. Always use Colab Secrets.

### Step 3 — Personalise Cell 3

Open Cell 3 and edit these variables to match your profile:

```python
YOUR_NAME     = "Your Name"
YOUR_ROLE     = "Data Scientist / ML Engineer"
YOUR_AUDIENCE = "data scientists, ML engineers, and tech professionals"
YOUR_TONE     = """
- Conversational and direct, not corporate or salesy
...
"""
```

The `TOPIC_CATEGORIES` list controls what the agent writes about. Add, remove, or reword topics to match your interests. The agent avoids repeating any category within a 7-day window.

### Step 4 — Run all cells in order

Run cells 1 through 8 top to bottom. If any cell throws an error, do not skip it — each cell depends on the previous one.

Expected output at the end: `✅ Email sent to your@gmail.com`

---

## Notebook structure

| Cell | What it does |
|---|---|
| 1 — Install | Installs `anthropic`, `google-genai`, `feedparser`, `requests`, `Pillow` |
| 2 — Secrets | Loads API keys and Gmail credentials from Colab Secrets |
| 3 — Profile | Your name, role, tone guidelines, and topic bank |
| 4 — Topic rotation | Picks today's topic; saves history to Google Drive to avoid repeats |
| 5 — News fetch | Pulls top AI/ML stories from Hacker News and recent arXiv papers |
| 6 — Generate post | Claude writes a 150–280 word LinkedIn post grounded in today's news |
| 7 — Generate image | Gemini Nano Banana 2 creates a 1280×720 PNG from the post content |
| 8 — Email | Sends draft text + image to your Gmail as HTML email with attachment |

---

## Cost

| Item | Cost per run |
|---|---|
| Claude (`claude-opus-4-7`) | ~$0.02 |
| Gemini Nano Banana 2 image | ~$0.03 |
| **Total** | **~$0.05/day (~$1.50/month)** |

To cut costs further, swap `claude-opus-4-7` for `claude-haiku-4-5-20251001` in Cell 6. Quality is still strong for short-form posts and cost drops by ~20x.

---

## Scheduling daily runs

**Colab Pro — Scheduled Notebooks:**
1. Open the notebook in Colab
2. Click ⚙️ (top right) → Manage scheduled runs
3. Set a daily schedule (7am recommended)
4. Make sure all secrets have Notebook access toggled ON

The notebook will run automatically and email you the draft each morning.

---

## How to post to LinkedIn

LinkedIn's API is not available for personal profile posting without an approved partner account. The recommended workflow is:

1. Open the email draft that arrives each morning
2. Read and tweak anything that doesn't sound like you
3. Save the attached `linkedin_post_image.png`
4. On LinkedIn: Start a post → upload the image → paste the text → post

Alternatively, use a LinkedIn-approved scheduler like **Buffer** (free tier available) to queue posts in advance.

---

## Tips for better results

**Better posts:** Paste 2–3 of your best-performing LinkedIn posts as few-shot examples directly into the system prompt in Cell 6. Claude will mirror your demonstrated style much more accurately than described style.

**Better images:** The more concrete and story-driven the post, the better Gemini's image will be. Posts with specific tools, scenarios, or before/after comparisons produce the most accurate visuals.

**Topic tuning:** After a few weeks, note which topics get the most engagement on LinkedIn and reweight your `TOPIC_CATEGORIES` list accordingly.

**Testing:** During testing, comment out the resize line in Cell 7 (`generated_image.resize(...)`) to skip the 1280×720 conversion and preview the raw Gemini output faster.

---

## Models used

| Model | Provider | Purpose |
|---|---|---|
| `claude-opus-4-7` | Anthropic | Post text generation |
| `gemini-3.1-flash-image-preview` | Google (Nano Banana 2) | Image generation |

---

## File structure

```
linkedin_agent_v2.ipynb   — main notebook
README.md                 — this file

Google Drive (auto-created at runtime):
MyDrive/linkedin_agent/
  topic_history.json      — rolling 30-day topic log to prevent repeats
```

---

## Troubleshooting

**`SecretNotFoundError`** — The secret name in Colab doesn't match exactly. Check spelling, capitalisation, and underscores. Make sure Notebook access is toggled ON (blue).

**`BadRequestError: credit balance too low`** — Add credits at [console.anthropic.com/settings/billing](https://console.anthropic.com/settings/billing). $5 lasts months for daily posts.

**`No image returned from Gemini`** — Your Gemini API key needs billing enabled. Go to [aistudio.google.com](https://aistudio.google.com) and check your account status.

**Gmail send failure** — You must use a Gmail App Password, not your regular Gmail password. Generate one at [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords) (requires 2FA to be enabled first).

**Cells out of order** — Always run from Cell 1 downward. Variables like `draft` and `todays_topic` are set in earlier cells and required by later ones.
