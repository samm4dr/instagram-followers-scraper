# Scrape Instagram Followers Without Login: Competitor Audiences as Lead Lists

Instagram's API won't give you follower lists — but follower lists are exactly what agencies and sellers want: who follows your competitor = who should follow you. This tutorial uses Apify actors (17k+ users on the top one) to pull followers, following, and profile data at scale — no cookies, no login.

> 🤖 **Try it live** — chat with a Poe bot that generates the exact pipeline for your niche: [Social Lead Finder](https://poe.com/socialleadfinder) · [Lead List Generator](https://poe.com/leadlistgenerator) · [Job Postings Miner](https://poe.com/jobpostingsminer) · [Amazon Product Miner](https://poe.com/amazonproductminer) · [SERP Rank Tracker](https://poe.com/serpranktracker)

## The stack

- **[Apify](https://console.apify.com/sign-up?fpr=dx06p)** — Instagram actors that handle rate limits + proxies. Free monthly credits.
- **Python** — followers → CSV → enrichment.

## Step 1 — Token

[Create an Apify account](https://console.apify.com/sign-up?fpr=dx06p) → Settings → Integrations → `export APIFY_TOKEN=...`

## Step 2 — Pull followers of any account

```bash
curl -X POST "https://api.apify.com/v2/acts/apify~instagram-followers-following-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"username": "competitor_account", "operation": "followers", "resultsLimit": 5000}'
```

Fields: `username`, `fullName`, `biography`, `externalUrl`, `followerCount`, `isBusinessAccount`, `verified`.

## Step 3 — Filter for leads

```python
from apify_client import ApifyClient

client = ApifyClient("APIFY_TOKEN")
run = client.actor("apify/instagram-followers-following-scraper").call(run_input={
    "username": "big_competitor", "operation": "followers", "resultsLimit": 5000,
})

leads = []
for u in client.dataset(run["defaultDatasetId"]).iterate_items():
    bio = (u.get("biography") or "").lower()
    if any(k in bio for k in ["founder", "owner", "ceo", "shop", "est."]) \
            or u.get("isBusinessAccount"):
        leads.append(u["username"])

print(len(leads), "business followers = warm audience")
```

## The plays

- **Agencies** — competitor followers who run businesses = your prospect list for SMM/branding pitches.
- **Dropshippers** — niche-page followers = lookalike audience for ads and DM outreach.
- **Growth checks** — scheduled follower-count runs (`apify/instagram-followers-count-scraper`, 16k users) = growth velocity intel per account.

## Useful actors

| Actor | Job | Users |
|---|---|---|
| `apify/instagram-followers-following-scraper` | Follower/following lists | 4.5k |
| `scraping_solutions/instagram-scraper-followers-following-no-cookies` | No-login follower pull | 17.9k |
| `apify/instagram-followers-count-scraper` | Fast count snapshots | 16.4k |
| `apify/instagram-profile-scraper` | Full profile data | huge |

## Automation

Weekly scheduled run → diff follower lists → who's new, who left. Combine with `vdrmota/contact-info-scraper` on `externalUrl` → emails for outreach.

## More resources

- [apify-skills](https://github.com/g4dr/apify-skills) — `npx skills add g4dr/apify-skills`, runs this from a prompt
- [awesome-web-scraping](https://github.com/g4dr/awesome-web-scraping) — curated tools + actors
- Free credits: [Apify signup](https://console.apify.com/sign-up?fpr=dx06p)
