<div align="center">

```
```
██████╗ ██╗   ██╗██████╗ ██╗   ██╗███████╗██╗  ██╗
██╔══██╗╚██╗ ██╔╝██╔══██╗██║   ██║██╔════╝██║  ██║
██████╔╝ ╚████╔╝ ██████╔╝██║   ██║███████╗███████║
██╔═══╝   ╚██╔╝  ██╔═══╝ ██║   ██║╚════██║██╔══██║
██║        ██║   ██║     ╚██████╔╝███████║██║  ██║
╚═╝        ╚═╝   ╚═╝      ╚═════╝ ╚══════╝╚═╝  ╚═╝
```

Raphael Guintoli
**Autonomous multi-platform content syndication — powered by LLM inference**

[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=flat-square&logo=openai&logoColor=white)](https://openai.com)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-active-22c55e?style=flat-square)]()

*Ingest → Curate → Summarize → Schedule → Post. Fully automated.*

</div>

---

## What is PyPush?

PyPush is a **zero-touch content pipeline** that monitors RSS feeds and news sources, uses an LLM to curate and semantically summarize the most relevant content, then cross-posts it across LinkedIn, Twitter/X, and Bluesky on a configurable schedule — all orchestrated by GitHub Actions with no server required.

You define your topics. PyPush handles everything else.

```
┌─────────────────────────────────────────────────────────────────┐
│                        PyPush Pipeline                          │
│                                                                 │
│  📡 RSS / APIs        🧠 LLM Curation         📤 Publishing    │
│  ─────────────        ──────────────          ─────────────     │
│  TechCrunch     ──►   Relevance filter  ──►   LinkedIn         │
│  HN / Reddit    ──►   Dedup & rank      ──►   Twitter / X      │
│  Custom feeds   ──►   Summarize         ──►   Bluesky          │
│                        Rewrite for             Telegram         │
│                        platform tone           (optional)       │
│                                                                 │
│                    ⏱  Scheduled via GitHub Actions CRON        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key features

| Feature | Details |
|---|---|
| **Multi-source ingestion** | RSS, Atom, HN API, Reddit, custom URLs |
| **LLM-powered curation** | GPT-4o filters noise, ranks by relevance score, deduplicates |
| **Semantic summarization** | Platform-aware rewrites — thread-style for X, professional tone for LinkedIn |
| **Scheduled CI/CD** | Runs on GitHub Actions cron — no server, no infra cost |
| **Rate-limit aware** | Respects API quotas, exponential backoff, dry-run mode |
| **Fully configurable** | YAML config for sources, topics, posting frequency, tone |
| **Audit log** | Every post logged with source URL, LLM score, and timestamp |

---

## Architecture

```
pypush/
├── ingestion/
│   ├── rss_reader.py        # Fetch & parse RSS/Atom feeds
│   ├── hn_scraper.py        # Hacker News top stories
│   └── deduplicator.py      # Hash-based dedup across runs
│
├── curation/
│   ├── ranker.py            # LLM relevance scoring (0–1)
│   ├── summarizer.py        # Platform-aware summarization
│   └── prompt_templates/    # Jinja2 prompts per platform
│
├── publishing/
│   ├── linkedin_poster.py   # LinkedIn API v2
│   ├── twitter_poster.py    # Twitter API v2
│   └── bluesky_poster.py    # AT Protocol
│
├── scheduler/
│   └── queue.py             # Priority queue + rate-limit manager
│
├── config.yaml              # Topics, sources, schedule, tone
├── run.py                   # Pipeline entrypoint
└── .github/workflows/
    └── syndicate.yml        # GitHub Actions cron job
```

---

## Quickstart

```bash
# 1. Clone
git clone https://github.com/YOUR_USERNAME/pypush
cd pypush

# 2. Install
pip install -r requirements.txt

# 3. Configure
cp config.example.yaml config.yaml
# → edit topics, feed URLs, and platform preferences

# 4. Set secrets (or add to GitHub repo secrets for CI)
export OPENAI_API_KEY="sk-..."
export LINKEDIN_ACCESS_TOKEN="..."
export TWITTER_BEARER_TOKEN="..."

# 5. Dry run — see what would be posted, nothing is published
python run.py --dry-run

# 6. Live run
python run.py
```

---

## Configuration

```yaml
# config.yaml
topics:
  - "AI in marketing"
  - "MarTech automation"
  - "generative AI for business"

sources:
  rss:
    - https://techcrunch.com/feed/
    - https://feeds.feedburner.com/venturebeat/SZYF
  hackernews:
    top_n: 20
    min_score: 100

curation:
  min_relevance_score: 0.72   # LLM relevance threshold (0–1)
  max_posts_per_run: 3
  dedup_window_days: 7

platforms:
  linkedin:
    enabled: true
    tone: "professional, insightful"
    max_length: 1200
  twitter:
    enabled: true
    tone: "punchy, opinionated"
    thread: false
  bluesky:
    enabled: false

schedule:
  cron: "0 8,12,17 * * 1-5"   # 3x/day on weekdays
```

---

## GitHub Actions — zero-infra deployment

```yaml
# .github/workflows/syndicate.yml
name: PyPush Syndication

on:
  schedule:
    - cron: "0 8,12,17 * * 1-5"
  workflow_dispatch:              # manual trigger from GitHub UI

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: python run.py
        env:
          OPENAI_API_KEY:           ${{ secrets.OPENAI_API_KEY }}
          LINKEDIN_ACCESS_TOKEN:    ${{ secrets.LINKEDIN_ACCESS_TOKEN }}
          TWITTER_BEARER_TOKEN:     ${{ secrets.TWITTER_BEARER_TOKEN }}
```

Push to `main` and you're live. GitHub runs the pipeline on schedule at no cost.

---

## LLM curation — how it works

Each ingested article is scored by GPT-4o against your configured topics:

```python
# Simplified scoring logic
prompt = f"""
Rate the relevance of this article to the topics: {topics}
Return a JSON: {{"score": float, "reason": str, "suggested_angle": str}}

Title: {article.title}
Summary: {article.description}
"""
# score >= 0.72 → queued for publishing
# score <  0.72 → discarded, logged
```

Top-ranked articles are then rewritten per platform — same story, different voice.

---

## Requirements

```
openai>=1.30
feedparser>=6.0
tweepy>=4.14
linkedin-api>=2.0
atproto>=0.0.46         # Bluesky AT Protocol
pyyaml>=6.0
jinja2>=3.1
httpx>=0.27
schedule>=1.2
```

---

## Roadmap

- [ ] Webhook support — trigger pipeline on Slack command
- [ ] Analytics module — track post engagement per source
- [ ] Vector store dedup — semantic similarity instead of hash
- [ ] Web UI for topic and schedule management
- [ ] Support for Instagram, Threads, Mastodon

---

## License

MIT — use it, fork it, build on it.

---

<div align="center">

*Built for marketers who think in systems, not tasks.*

</div>
