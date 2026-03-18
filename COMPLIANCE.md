# Compliance & Terms of Service

## Disclaimer

This project is published for **educational and personal use purposes**. It demonstrates how AI agents can automate job search workflows, including interaction with job platforms and career websites.

This repository accesses job platforms in two ways:

1. **Node.js scrapers** (`scripts/scrapers/`) that make direct HTTP requests to public APIs and RSS feeds. These are used only for sources that explicitly provide programmatic access.

2. **Natural language instructions** (skill and agent definitions in `.claude/`) that tell Claude Code to open job platforms in your own Chrome browser (via the [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome/) extension), using your existing logged-in accounts and authentication. It navigates pages, scrolls through listings, and reads job postings — the same way you would browse manually. No headless browsers, no fake accounts, and no direct HTTP scraping — all browsing happens in your real Chrome session.

Some of the platforms accessed via browser automation may restrict or prohibit automated access in their Terms of Service. The browser automation instructions in this repository are provided as-is for learning and reference. **By using this project, you accept full responsibility for ensuring your use complies with all applicable laws, regulations, and platform Terms of Service in your jurisdiction.**

The authors and contributors of this project are not responsible for any misuse or any consequences arising from the use of this software. See the [LICENSE](LICENSE) file for full warranty and liability disclaimers.

## How Each Source Is Accessed

### Public API / Open Data (Node.js scrapers)

These sources provide explicit programmatic access. The repository includes Node.js scripts that call them directly.

| Source | Method | Details |
|--------|--------|---------|
| **Arbeitnow** | REST API | Paginated JSON endpoint (`/api/job-board-api`). Rate-limited with delays between pages. Their API terms require attribution — a link back to [arbeitnow.com](https://www.arbeitnow.com). They reserve the right to revoke API access at any time. |
| **SimplifyJobs** | GitHub raw file | Reads a publicly maintained JSON file from the [SimplifyJobs](https://github.com/SimplifyJobs/New-Grad-Positions) GitHub repository. Single HTTP request per run. Note: the repo has no LICENSE file (default copyright applies), and Simplify.jobs' main site ToS prohibits scraping. However, the repo is public, community-contributed, and the README encourages derivative projects. |
| **Secret Tel Aviv** | RSS feed | Standard XML/RSS feed (`/wpjobboard/xml/rss/`). Fetched at most a few times per day. No published ToS; robots.txt permits access. |

### Browser Automation (Claude in Chrome)

These sources do not offer public APIs. Instead of scraping them, the repository contains instructions that tell Claude Code to browse them in your own Chrome browser, using your existing accounts — navigating, scrolling, and reading content like a human user.

| Source | What the instructions do |
|--------|--------------------------|
| **LinkedIn** | Search for jobs by keyword/location, scroll through result pages, read job cards and descriptions |
| **AllJobs** | Browse job listings, scroll to load content, read postings |
| **Built In** | Navigate to location-filtered pages, read job listings |
| **Wellfound** | Browse startup job listings, read postings |

The browser automation runs at normal browsing speed with human-like pauses. It uses your own Chrome browser and your own accounts — no headless browsers, no fake accounts, no bypassing of authentication or rate-limiting mechanisms.

### WhatsApp Groups (optional, self-hosted API)

Job postings shared in WhatsApp groups can be read via a WhatsApp skill (e.g., [claude-code-whatsapp](https://github.com/roysahar11/claude-code-whatsapp) which uses [WAHA](https://waha.devlike.pro/), a self-hosted WhatsApp Web API). This reads the user's own chat history from their own WAHA instance — no third-party access is involved. WhatsApp integration is optional; the pipeline works without it.

## Respectful Access Principles

- **Rate limit** all automated requests — scrapers include delays between pages
- **Minimize volume** — fetch only what you need for your personal search
- **Respect robots.txt** where applicable
- **Stop if blocked** — if a platform rate-limits or blocks you, don't circumvent it

## Adding New Sources

Before automating a new job source, check:

1. Does it offer a public API or RSS feed? → Use a direct scraper
2. What does its Terms of Service say about automated access?
3. What does its robots.txt permit?

Use direct HTTP scrapers only for sources that explicitly permit programmatic access. For everything else, use browser automation with awareness of the associated ToS considerations.
