# Appliable Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [0.1.2] - 2026-04-05

### Quality Improvements
- **customize-resume**: Comprehensive instructions for writing better resume summaries — structured process for understanding the role's real needs and crafting a compelling, genuine pitch. Improved guidance for role title alignment, business-value framing, and highlight selection.
- **personal-note**: Added nuances to personal note writing — addressing requirement gaps honestly, relocation context for international roles, more natural closing

### Performance
- Added `ultrathink` and max effort directives across agents — fixes a quality drop since Claude Code introduced effort settings

### Tools for Contributors
- **publish skill**: Diff-based incremental publishing — changed files are patched via git diff instead of re-sanitized from scratch, preserving approved sanitization decisions between publishes

## [0.1.1] - 2026-03-19

### Skills
- **quick-apply**: Enhanced WhatsApp notifications — draft PDF sent with caption (verdict, URL, base resume score, key customizations), base resume PDF sent back-to-back for comparison, source field from pipeline data
- **daily-job-fetch**: Scan-jobs prompts now reference profile files instead of hardcoded summaries; removed hardcoded language/remote preferences (read from evaluation-criteria.md)
- **publish**: Added publish/SKILL.md to Product tier, scripts/daily-job-fetch.sh to Ignored list

### Agents
- **job-description-fetcher**: Language config now references config/search.md instead of hardcoded values
- **scan-jobs**: Generalized all user references for better reusability
- **quick-apply-batch**, **job-fetcher**: Minor generalization fixes

### Scripts
- Minor formatting fixes in create-quick-apply-batches.js, merge-pipeline.js, arbeitnow.js

## [0.1.0] - 2026-03-18

Initial public release.

### Skills
- `/setup` — Guided onboarding that builds your profile through conversation
- `/daily-job-fetch` — Full pipeline: fetch from multiple sources, scan, report, draft applications
- `/customize-resume` — Tailored resume generation with auto-layout optimization and PDF rendering
- `/quick-apply` — Autonomous batch application drafting with WhatsApp delivery
- `/scan-jobs` — Two-pass job evaluation: relevance judgment + resume scoring
- `/linkedin-job-fetch` — LinkedIn job extraction via browser automation
- `/personal-note` — Cover letters grounded in documented experience
- `/publish` — Contribute improvements back with automatic PII sanitization

### Agents
- `job-fetcher` — Multi-source job collection (LinkedIn, WhatsApp, Chrome sources)
- `job-description-fetcher` — Two-tier description retrieval with content validation and language detection
- `scan-jobs` — Profile-based relevance analysis with resume scoring
- `quick-apply-batch` — Batch resume customization with quality iteration

### Scripts
- `merge-pipeline.js` — Multi-source merge with location-aware deduplication
- `create-quick-apply-batches.js` — Batch creation with multi-location grouping
- `filter-by-urls.js` — URL-based pipeline filtering
- Web scrapers: Secret Tel Aviv (RSS), Arbeitnow (API), SimplifyJobs (GitHub)

### Infrastructure
- Profile system: context, experience, preferences, evaluation criteria, resume preferences, coaching notes
- Config system: user identity, search parameters, source configuration
- Application tracking via directory structure with status files
- Resume rendering pipeline: JSON → HTML → auto-fit → PDF
