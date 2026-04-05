---
name: publish
description: Contribute improvements back to the project. Syncs your private repo changes to a fork, automatically stripping personal data so you can open a clean PR.
disable-model-invocation: true
argument-hint: [optional: path to target repo]
---

# Publish to Public Repo

Easily contribute your improvements back to the community. This skill syncs product files from your private repo to a fork of the upstream project, automatically stripping personal data, generating clean example templates, and presenting a diff for review before committing. The result is a clean fork you can open a PR from.

Built new skills, added job sources, improved workflows? This is how you share them.

## Step 0: Load Config + Validate

1. Read `config/publish.md` for the target repo path (or use `$ARGUMENTS` if a path was provided)
2. Verify the target repo exists and is a git repo (`git -C "{TARGET_PATH}" status`)
3. If the target repo has a dirty working tree, warn and ask whether to proceed or abort

Store `TARGET_PATH` for use in all subsequent steps.

## Step 0.5: Change Detection

Identify which files actually changed since the last publish so unchanged files can be skipped.

1. Run `git ls-tree -r HEAD` in the private repo to get the current blob SHA for every tracked file
2. Load `state/publish-manifest.json` (if it exists). If missing or corrupted, treat all files as changed (full publish).
3. For each file that will be processed in Steps 1 and 2, compare its current blob SHA to the manifest entry:
   - **Changed**: blob SHA differs from manifest, or file not in manifest (new file) → process normally
   - **Unchanged**: blob SHA matches manifest → skip processing
4. Check for **deletions**: files in the manifest that no longer exist in `git ls-tree` (deleted from private repo). These need cleanup in the target repo via Step 3.
5. Build a skip list and report it: "Skipping N unchanged files: {list}"
6. If ALL files are unchanged AND no deletions detected, report "Nothing changed since last publish" and stop

The manifest only tracks Product and Template source files — Protected and Ignored files are never in the manifest.

### Publish modes

Based on the manifest state, this publish runs in one of two modes:

| Mode | When | Behavior |
|------|------|----------|
| **Incremental** | Manifest exists with a valid `commitRef` | Changed files are updated via diff-based patching (Steps 0.6, 1, 2) |
| **Full** | No manifest, corrupted manifest, or first publish | All files are processed from scratch with full sanitization |

In incremental mode, individual files still fall back to fresh sanitization when they have no existing public version (new files).

## Step 0.6: Collect and Understand Diffs (incremental mode only)

Skip this step entirely in full publish mode.

Before processing any files, build a complete picture of what changed:

1. For every changed Product and Template file, collect the private diff:
   ```bash
   git diff {commitRef}..HEAD -- "{file_path}"
   ```
   Use the `commitRef` from the manifest. Use sufficient context lines (default 3 is usually enough; use `-U10` if a diff feels ambiguous).

2. Read all collected diffs together. Understand the full change set:
   - What was the intent of the changes? (new feature, bug fix, refactor, content update)
   - Do any changes reference each other across files?
   - Are there new PII patterns introduced that will need sanitization?

3. Report a brief summary: "N files changed since last publish ({commitRef}): {grouped summary of changes}"

This understanding informs how changes are applied in Steps 1 and 2.

## Step 1: Copy Product Files (with Inline PII Sanitization)

Walk the private repo and classify every file into one of four tiers. This is dynamic — new files are automatically discovered and classified using the tier principles below.

### File Tiers

| Tier | Principle | Examples |
|------|-----------|---------|
| **Product** | Reusable workflow logic, skills, agents, scripts, project config | `.claude/skills/*` (including `publish/SKILL.md`), `.claude/agents/*`, `scripts/*`, `CLAUDE.md`, `package.json`, `package-lock.json`, `.claude/settings.json` |
| **Template** | Personal data files that new users need their own version of | `profile/*.md`, `config/user.md`, `config/search.md` |
| **Protected** | Target-repo-only files maintained independently — never overwritten | `LICENSE`, `COMPLIANCE.md`, `CONTRIBUTING.md`, `Resumes/README.md`, `.git/`, `.gitignore`, `.claude/skills/setup/`, `*.example.md`, `README.md` |
| **Ignored** | Operational data, personal state, runtime artifacts | `Applications/`, `Resumes/*.json`, `state/`, `logs/`, `Reports/`, `research/`, `.claude/settings.local.json`, `.claude/memory/`, `.claude/plans/`, `.claude/ideas/`, `config/publish.md`, `state/publish-manifest.json`, `node_modules/`, `.DS_Store`, `._*`, `scripts/daily-job-fetch.sh` |

When encountering a file not listed above, classify it by its principle:
- Reusable workflow logic → Product
- User-specific data others need to create → Template
- Target-repo-only → Protected
- Operational/personal/runtime → Ignored
- When unsure → ask the user

### Sanitization

For each **Product** file, read it and separate personal details from workflow logic. Generalize personal content while preserving the structural and behavioral instructions that make the system work.

**What to generalize:**
- Names, contact info, phone numbers, emails, LinkedIn/GitHub URLs
- Personal anecdotes and specific life details
- Specific company names from your experience used as examples
- WhatsApp group IDs and names
- LinkedIn geoIds and location-specific search parameters
- Any content that identifies a specific person

**What to preserve exactly:**
- Workflow steps, process instructions, and behavioral rules
- Architecture decisions and technical patterns
- File path conventions and directory structure references
- Tool usage patterns (MCP tools, Chrome automation, WhatsApp skill invocations)
- Error handling logic and fallback strategies
- References to config/profile files (these become the user's own files)
- Geographic/product scope references (e.g., "Israeli pipeline", "international scrapers") — these describe product architecture and feature scope, not personal data
- All emojis, special symbols, and unicode characters — preserve them exactly as they appear

**Approach:** Don't follow a rigid find-and-replace. Read each file, understand the content contextually, and decide what needs changing. A mention of a person's name in a workflow instruction becomes "the user" or gets removed. A personal example becomes a generic one. A hardcoded WhatsApp group ID becomes a reference to `config/search.md`.

**Mixed files — extraction over generalization:** Before generalizing personal content in a product file, ask: "Is this a personal *preference* that other users would want to customize differently?" If yes, extraction to a personal file is better than generalization — it preserves the customization point for all users.

Signals that content should be extracted rather than generalized:
- Instructions about what to always/never include (content emphasis rules)
- Cultural, regional, or market-specific rules
- Personal examples that map non-standard experience to professional skills
- Any "how I want my [X] done" instruction as opposed to "how [X] works mechanically"

When extraction is identified: propose it to the user before proceeding.

### Agent file naming

Agent definition files use lowercase `agent.md` in the target repo (regardless of casing in private repo).

### Copy process

For each Product file **not on the skip list from Step 0.5**:

**Incremental mode** (manifest exists and file has a public version):
1. Read the file's diff from Step 0.6
2. Read the current public version of the file from the target repo
3. Apply the equivalent sanitized change to the public version — the diff shows the raw private content that changed, and the existing public file shows how previous content was sanitized. Follow the same sanitization patterns already established in the file.
4. Write the updated file to the target repo

**Fresh sanitization** (full publish mode, OR file is new with no public version):
1. Read the file from the private repo
2. Apply sanitization (contextual, not mechanical) using the rules above
3. Write the sanitized version to the same relative path in the target repo
4. Create any needed directories

## Step 2: Generate `.example.md` Templates

For each **Template** file **not on the skip list from Step 0.5**, generate or update a template version:

| Source (private) | Destination (target) |
|-----------------|---------------------|
| `profile/context.md` | `profile/context.example.md` |
| `profile/experience.md` | `profile/experience.example.md` |
| `profile/preferences.md` | `profile/preferences.example.md` |
| `profile/coaching-notes.md` | `profile/coaching-notes.example.md` |
| `profile/evaluation-criteria.md` | `profile/evaluation-criteria.example.md` |
| `profile/resume-preferences.md` | `profile/resume-preferences.example.md` |
| `config/user.md` | `config/user.example.md` |
| `config/search.md` | `config/search.example.md` |

**Incremental mode** (manifest exists and `.example.md` exists in target repo):
1. Read the file's diff from Step 0.6
2. Read the current `.example.md` from the target repo
3. Classify each change as **structural** or **personal content**:
   - **Structural**: sections added/removed, new entry types, format patterns changed → apply to template automatically
   - **Personal content**: the user's data changed but structure stayed the same → present to the user and ask if the template examples should be updated to reflect the new direction
4. Apply structural changes. For personal content changes, present the list and let the user decide.
5. Write the updated template (or skip if no changes needed)

**Fresh generation** (full publish mode, OR template doesn't exist yet):

**Template generation rules:**
- Preserve all headings and structural format exactly
- Replace personal content with placeholder instructions that explain what to put there
- Include examples of the format expected (e.g., "Your Name" not just "fill in name")
- Templates must be immediately usable — copy `.example.md` to `.md`, fill in your details, and the system works
- Don't include any of the original personal data in the templates

## Step 3: Auto-Clean Stale Files

Delete files in the target repo that don't exist in the private repo AND aren't Protected.

1. Build a manifest of all expected files (from Steps 1 and 2)
2. Walk the target repo, skipping Protected files and directories (`.git/`, `LICENSE`, `README.md`, `CONTRIBUTING.md`, `.gitignore`, `Resumes/README.md`, `.claude/skills/setup/`, `*.example.md`)
3. Delete anything not in the manifest and not Protected
4. If >10 files would be deleted, warn and ask for confirmation before proceeding
5. Report what was deleted (or "No stale files found")

## Step 4: Update README.md

Read the existing `README.md` in the target repo (if it exists), then read the current project state from the files copied in Steps 1-2.

**If README.md exists:** Update it to reflect any changes since the last publish — add new features, remove deleted ones, update descriptions. Preserve existing wording and structure where nothing changed.

**If README.md doesn't exist (first run):** Generate an initial version covering project overview, features, architecture, prerequisites, getting started, pipeline workflow, file structure, configuration, compliance, and license.

## Step 5: PII Verification (before commit)

Run grep checks on the target repo **before committing** — PII should never be committed to a public repo.

Build grep patterns dynamically from `config/user.md`:
- Extract the user's full name, phone number(s), email(s), and any other identifying info
- Search for these patterns across all text files in the target repo

```bash
grep -ri "{NAME}\|{PHONE}\|{EMAIL}" "{TARGET_PATH}/" --include="*.md" --include="*.js" --include="*.json" --include="*.html" --include="*.sh" --include="*.yaml" --include="*.yml"
grep -ri "@g.us" "{TARGET_PATH}/" --include="*.md" --include="*.js" --include="*.json"
```

**Known exceptions** (not PII leaks):
- LICENSE file may contain the author name
- Links to public GitHub repos are intentional public references

**If PII found:** Fix the files immediately, then re-run verification until clean.
**If clean:** Report "PII check passed" and proceed to Step 6.

## Step 6: Cross-Check and Present Diff for Review

```bash
cd "{TARGET_PATH}" && git add -A && git diff --cached --stat
```

**Cross-check (incremental mode):** Before presenting to the user, verify that the public-side diff reflects the same intent as the private diffs collected in Step 0.6. For each changed file, confirm:
- The public diff captures the equivalent change (accounting for sanitization)
- No changes were accidentally dropped or misapplied
- No unchanged parts of the public file were inadvertently modified

If any discrepancy is found, fix it before presenting.

**Present to user:** Show the summary. If the diff is large, summarize changes by directory. Then ask: "Review the changes. Should I commit and push?"

For the first publish, also highlight the key sanitized files and ask the user to spot-check PII-sensitive ones (CLAUDE.md, customize-resume, job-fetcher, personal-note).

## Step 7: Commit and Push (on approval)

Generate a detailed commit message from the diff.

**Format:**
```
Sync: {one-line summary}

{Grouped list of changes by category — what was updated, added, removed, and why}
```

```bash
cd "{TARGET_PATH}" && git commit -m "{message}" && git push
```

After a successful commit, write `state/publish-manifest.json` in the **private** repo with the current blob SHAs for all Product and Template source files:

```json
{
  "publishedAt": "2026-03-11T14:30:00Z",
  "commitRef": "abc1234",
  "files": {
    "CLAUDE.md": "a1b2c3d...",
    "scripts/scrapers/fetch-all.js": "e4f5g6h..."
  }
}
```

- `commitRef`: HEAD of the private repo at time of publish
- `files`: map of relative path → blob SHA from `git ls-tree`
- Include ALL Product and Template source files, not just the ones that changed this run
- Deleted files naturally drop from the manifest

## Notes

- The skill is idempotent — running it multiple times produces the same result
- Agent filenames are normalized to lowercase `agent.md` in the target repo
- If new `profile/` or `config/` files are added later, update the template table in Step 2
- The skill never modifies the private repo (except writing the manifest to `state/`)
