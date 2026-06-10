# xtool — X (Twitter) Scraping Toolkit

Scrapes liked tweets and bookmarks from X into structured JSON. Combines live DOM scraping with intercepted GraphQL network responses to capture full text, expanded URLs, media, quote tweets, and author data.

---

## Setup

```bash
cd /Users/mp/Desktop/twitter-scraping/twitter-extraction
python3 -m venv venv_playwright
source venv_playwright/bin/activate
pip install playwright browser-cookie3 pyobjc-framework-Cocoa
playwright install chromium
```

> Chrome must be fully closed before running any command. `browser-cookie3` reads the Cookies file directly from disk and will fail or return stale data if Chrome has it locked.

> Terminal needs Full Disk Access enabled in System Settings → Privacy & Security → Full Disk Access for cookie extraction to work.

---

## Configuration

All user-specific settings are in `config.py`:

```python
CHROME_PROFILE = Path("/Users/mp/Library/Application Support/Google/Chrome/Profile 1")
X_USERNAME     = "peers8862"
```

If you use a different Chrome profile, open `chrome://version` in Chrome and check the **Profile Path** field.

---

## First-time setup

After installing dependencies, run the setup wizard to configure your X username and Chrome profile:
```bash
python xtool.py start
```
This scans your Chrome profiles, lets you pick the right one, and writes `config.py` automatically.

---

## Commands

Always activate the venv first:
```bash
source venv_playwright/bin/activate
```

### Scrape likes
```bash
python xtool.py likes                  # incremental — only fetches newer than last run
python xtool.py likes --limit 100      # force collect exactly 100 (ignores cutoff)
```

### Scrape bookmarks
```bash
python xtool.py bookmarks              # incremental — only fetches newer than last run
python xtool.py bookmarks --limit 500  # force collect exactly 500
```

### Audit
Checks the dataset for tweets with missing links or media and writes a flagged list for enrichment.
```bash
python xtool.py audit --type likes
python xtool.py audit --type bookmarks
```

### Scrape missing
Visits each flagged tweet's page individually to recover missing data. Run after `audit`.
```bash
python xtool.py scrape-missing --type likes
python xtool.py scrape-missing --type bookmarks
```

### Enrich
Looks for links posted by the original author in their own reply thread — useful for tweets where the link is in a follow-up reply rather than the original tweet.
```bash
python xtool.py enrich --type likes
python xtool.py enrich --type bookmarks
```

---

## Typical routine workflow

```bash
python xtool.py likes
python xtool.py bookmarks
```

That's it for a normal run. The incremental logic automatically detects the newest tweet already stored and stops when it reaches it — no manual limit needed.

To force a full scrape from scratch, delete the data file first:
```bash
rm data/x_likes.json
python xtool.py likes
```

For a full data quality pass:
```bash
python xtool.py audit --type likes
python xtool.py scrape-missing --type likes
python xtool.py enrich --type likes
```

---

## Output

| Path | Description |
|---|---|
| `data/x_likes.json` | Canonical likes dataset, prepended on each run |
| `data/x_bookmarks.json` | Canonical bookmarks dataset, prepended on each run |
| `exports/likes_YYYY-MM-DD_HHMM.json` | Timestamped snapshot saved after every likes run |
| `exports/bookmarks_YYYY-MM-DD_HHMM.json` | Timestamped snapshot saved after every bookmarks run |
| `data/x_likes_audit_report.txt` | Last audit report for likes |
| `data/x_bookmarks_audit_report.txt` | Last audit report for bookmarks |
| `debug_output/` | Screenshots, HTML snapshots, and tracebacks on failure |

---

## Output schema

Each tweet record:
```json
{
  "url": "https://x.com/user/status/123",
  "text": "full tweet text",
  "author": { "name": "Display Name", "handle": "@username" },
  "timestamp": "2025-04-01T12:00:00.000Z",
  "links": {
    "urls": ["https://example.com/article"],
    "mentions": ["https://x.com/someuser"]
  },
  "media": [
    { "type": "photo", "url": "https://pbs.twimg.com/..." }
  ],
  "quote": {
    "tweetId": "456",
    "text": "quoted tweet text",
    "author": "@quoteduser",
    "timestamp": "2025-03-30T10:00:00.000Z",
    "links": ["https://example.com/other"]
  }
}
```

---

## Project structure

```
twitter-extraction/
├── xtool.py              ← CLI entry point
├── config.py             ← user config and paths
├── core/
│   ├── cookies.py        ← Chrome cookie extraction
│   ├── graphql.py        ← GraphQL response parser
│   ├── dom.py            ← DOM scraper (Playwright page.evaluate)
│   └── merge.py          ← classify_links, parse_author, merge
├── commands/
│   ├── likes.py          ← likes scrape logic
│   ├── bookmarks.py      ← bookmarks scrape logic
│   ├── audit.py          ← audit command
│   ├── scrape_missing.py ← scrape-missing command
│   └── enrich.py         ← enrich command
├── data/                 ← canonical JSON outputs and audit files
├── exports/              ← timestamped snapshots per run
├── debug_output/         ← failure screenshots and tracebacks
├── original_scripts/     ← original standalone scripts (archived)
└── lessons_learned.md    ← architecture notes and scraping lessons
```

---

## How it works

Each scrape combines two data sources:

- **DOM** — establishes tweet identity (URL → tweet ID), captures visible text and author
- **GraphQL** — intercepted network responses provide full text, expanded URLs, media objects, and quote tweet data that isn't fully rendered in HTML

The `url_map` and `full_text_map` dictionaries bridge the two. DOM snapshots are collected first, then merged with GraphQL data after the scroll loop completes — ensuring all network responses have arrived before merging.

t.co shortlinks are stripped unconditionally. Profile mention URLs are separated from resource URLs into `links.mentions`. Long-form tweets prefer `note_tweet` over `legacy.full_text`.

For a deeper dive see `lessons_learned.md`.
