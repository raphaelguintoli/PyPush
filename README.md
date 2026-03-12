# 📰 News Auto-Poster — Free Edition

Automatically scrapes news articles, generates AI comments using **Groq (free Llama 3)**, and posts to **X** and **Bluesky** on a schedule via **GitHub Actions** — all 100% free.

---

## 💰 What's Free & Why

| Component | Service | Cost |
|---|---|---|
| AI comment generation | Groq API (Llama 3) | ✅ Free tier |
| Automation & scheduling | GitHub Actions | ✅ Free tier (2000 min/month) |
| Posting to Bluesky | Bluesky AT Protocol | ✅ Always free |
| Posting to X (Twitter) | Twitter API v2 Free | ✅ Free (1,500 posts/month) |

> Running once per hour = ~720 posts/month, well within all free limits.

---

## 🚀 Setup (5 steps)

### Step 1 — Fork this repository

Click **Fork** on GitHub to copy this repo to your account.

### Step 2 — Get your Groq API key (free, no credit card)

1. Go to [console.groq.com](https://console.groq.com)
2. Sign up with Google or email
3. Click **API Keys → Create API Key**
4. Copy the key

### Step 3 — Get your X (Twitter) API keys

1. Go to [developer.twitter.com](https://developer.twitter.com)
2. Create a new app (free account)
3. Set app permissions to **Read and Write**
4. Go to **Keys and Tokens** → generate all four keys

### Step 4 — Get your Bluesky App Password

1. Open Bluesky → **Settings → Privacy and Security → App Passwords**
2. Click **Add App Password**, name it (e.g. "news-poster"), copy it

### Step 5 — Add Secrets to GitHub

In your repo: **Settings → Secrets and variables → Actions → New repository secret**

Add these 7 secrets:

| Secret Name | Where to get it |
|---|---|
| `GROQ_API_KEY` | console.groq.com |
| `X_API_KEY` | developer.twitter.com |
| `X_API_SECRET` | developer.twitter.com |
| `X_ACCESS_TOKEN` | developer.twitter.com |
| `X_ACCESS_TOKEN_SECRET` | developer.twitter.com |
| `BLUESKY_HANDLE` | Your handle, e.g. `you.bsky.social` |
| `BLUESKY_APP_PASSWORD` | Bluesky → Settings → App Passwords |

---

## ⚙️ Configure Your News Sites

Edit `config/sites.json` to set which websites to scrape:

```json
{
  "sites": [
    {
      "name": "My Site",
      "url": "https://example.com/news",
      "article_selector": "article",
      "title_selector": "h2",
      "link_selector": "a",
      "excerpt_selector": "p.summary",
      "keywords": ["AI", "tech"]
    }
  ]
}
```

**How to find the right CSS selectors:**
1. Open the target website in Chrome/Firefox
2. Press **F12** to open DevTools
3. Click the **inspector cursor** (top-left of DevTools)
4. Click on an article title → note the HTML tag and class
5. Use that as your selector (e.g. `h3.article-title`)

**keywords** is optional — leave it as `[]` to post all articles, or add words to filter (e.g. only articles mentioning "AI").

---

## 🕐 Adjust the Schedule

Edit `.github/workflows/news-poster.yml`:

```yaml
- cron: "0 * * * *"    # Every hour (default)
- cron: "0 */2 * * *"  # Every 2 hours
- cron: "0 9 * * *"    # Once daily at 9am UTC
- cron: "0 9,17 * * *" # 9am and 5pm UTC
```

Use [crontab.guru](https://crontab.guru) to build custom schedules.

---

## 🧪 Test Without Posting

Go to **Actions → News Auto-Poster (Free) → Run workflow** and set `dry_run = true`. This prints what would be posted without actually publishing anything.

---

## 📁 File Structure

```
news-autoposter-free/
├── .github/workflows/
│   └── news-poster.yml       # GitHub Actions schedule & config
├── config/
│   └── sites.json            # Your news sites configuration
├── src/
│   ├── main.py               # Orchestrator
│   ├── scraper.py            # CSS-selector web scraper
│   ├── ai_comment.py         # Groq / Llama 3 comment generator
│   ├── poster_x.py           # X (Twitter) API v2
│   └── poster_bluesky.py     # Bluesky AT Protocol
├── posted_articles.json      # Auto-managed deduplication cache
├── requirements.txt
└── README.md
```

---

## 🔧 Environment Variables

| Variable | Default | Description |
|---|---|---|
| `DRY_RUN` | `false` | Print posts without publishing |
| `MAX_POSTS_PER_RUN` | `3` | Max new articles per run |
| `POST_TO_X` | `true` | Enable/disable X posting |
| `POST_TO_BLUESKY` | `true` | Enable/disable Bluesky posting |

---

## ❓ Troubleshooting

**"0 articles found"** — Your CSS selectors are probably wrong. Open the site in DevTools and double-check them.

**X post fails** — Make sure your app has **Read & Write** permissions and that you regenerated the Access Token *after* changing permissions.

**Groq error** — Check your `GROQ_API_KEY` secret is set correctly. Visit [console.groq.com](https://console.groq.com) to verify your key.

**Same articles reposted** — The `posted_articles.json` cache is committed back to the repo by the Action. Make sure the workflow has `contents: write` permission (already set by default in this repo).

---

## License

MIT
