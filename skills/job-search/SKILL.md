---
name: job-search
description: Search for jobs matching my resume and preferences
argument-hint: "keyword to search"
---

# Job Search Skill

> **Priority hierarchy**: See `shared/references/priority-hierarchy.md` for conflict resolution.

Automated daily job search using browser automation.

## Quick Start

- `/proficiently:job-search` - Run daily search with default terms from matching rules
- `/proficiently:job-search AI infrastructure` - Search with specific keywords

## File Structure

```
scripts/
  evaluate-jobs.md     # Subagent for parallel job evaluation
assets/
  templates/           # Format templates (committed)
```

## Data Directory

Resolve the data directory using `shared/references/data-directory.md`.

---

## Workflow

### Step 0: Check Prerequisites

Resolve the data directory, then check prerequisites per `shared/references/prerequisites.md`. Resume and preferences are both required.

### Step 1: Load Context

Read these files:
- `DATA_DIR/resume/*` (candidate profile)
- `DATA_DIR/preferences.md` (preferences)
- `DATA_DIR/job-history.md` (to avoid duplicates)
- `DATA_DIR/linkedin-contacts.csv` (if it exists — for network matching)

Extract search terms from:
1. `$ARGUMENTS` if provided
2. Target roles from preferences

### Step 2: Browser Search — hiring.cafe

Use Claude in Chrome MCP tools per `shared/references/browser-setup.md`, navigating to https://hiring.cafe. For each search term, enter the query and apply relevant filters (date posted, location, etc.).

**Extracting results — IMPORTANT:** Do NOT use `get_page_text` on hiring.cafe or any large job listing page. It returns the entire page content and will blow out the context window.

Instead, extract job listings using `javascript_tool` to pull only structured data:

```javascript
// Extract visible job listing data from the page
Array.from(document.querySelectorAll('[class*="job"], [class*="listing"], [class*="card"], tr, [role="listitem"]'))
  .slice(0, 50)
  .map(el => el.innerText.trim())
  .filter(t => t.length > 20 && t.length < 500)
  .join('\n---\n')
```

If that selector doesn't match, take a screenshot to understand the page structure, then write a targeted JS selector for the specific site. The goal is to extract just the listing rows (title, company, location, salary) — never the full page.

As a fallback, use `read_page` (NOT `get_page_text`) and scan for listing elements.

**Note:** Hiring.cafe is just our search tool. Don't share hiring.cafe links with the user — you'll resolve direct employer URLs for the top matches in Step 6.

### Step 3: Browser Search — LinkedIn

LinkedIn is the second search source. It requires the user to be signed into LinkedIn in Chrome. Navigate to the jobs search URL below — if a login wall appears instead of results, skip LinkedIn entirely and note it to the user (do not attempt to sign in; account authentication is the user's job).

**Search via URL** (in the same tab or a new one):

```
https://www.linkedin.com/jobs/search/?keywords=KEYWORDS&location=LOCATION
```

Useful filter params to append:
- `f_E=1%2C2` — entry/associate experience level
- `f_TPR=r604800` — posted in past week (`r2592000` for past month)
- `f_WT=2` — remote

**Extracting results — IMPORTANT:** NEVER use `get_page_text` on LinkedIn. Its pages are enormous and will blow out the context window. Extract listings with `javascript_tool` over the job cards:

```javascript
// Each card is li[data-occludable-job-id]; the attribute is the job id.
// innerText's first ~5 lines give title/company/location/salary/badges.
Array.from(document.querySelectorAll('li[data-occludable-job-id]'))
  .map(li => li.getAttribute('data-occludable-job-id') + '|' +
    li.innerText.trim().split('\n').slice(0, 5).join(' | '))
  .join('\n')
```

The results list is virtualized — cards unload as you scroll. Scroll the results pane (the left column, not the window), then re-collect, accumulating into a window-scoped Map keyed by job id (e.g. `window.__jobs = window.__jobs || new Map()`) so nothing is lost. Results paginate at ~25 per page; advance pages as needed for more results.

**Job URLs**: each job's canonical URL is `https://www.linkedin.com/jobs/view/JOB_ID` — this is the URL to record and share.

**Apply routing**: an "Easy Apply" badge means the application is hosted on LinkedIn itself (see the LinkedIn Easy Apply pattern in `shared/references/ats-patterns.md`). Otherwise the Apply button leads to the employer's ATS — resolve and prefer the direct employer URL, as with hiring.cafe results.

**Dedupe**: before evaluation, dedupe LinkedIn results against the hiring.cafe results by company + title. Surviving LinkedIn jobs join the same fit-scoring and `job-history.md` flow as everything else.

### Step 4: Evaluate Jobs

Score each job against the candidate's resume and preferences using the criteria in `shared/references/fit-scoring.md`.

### Step 5: Save History

Append ALL jobs to `DATA_DIR/job-history.md`:

```markdown
## [DATE] - Search: "[terms]"

| Job Title | Company | Location | Salary | Fit | Notes |
|-----------|---------|----------|--------|-----|-------|
| ... | ... | ... | ... | ... | ... |
```

### Step 6: Resolve Employer URLs & Save Top Postings

For each **High-fit** job:
1. Click through the hiring.cafe listing to reach the actual employer careers page. For LinkedIn jobs: open `https://www.linkedin.com/jobs/view/JOB_ID` — if it's Easy Apply, that IS the application URL; otherwise follow the Apply button to the employer ATS and use that URL.
2. Capture the direct employer URL for the job posting
3. Extract the job description using `javascript_tool` to pull the posting content (e.g. `document.querySelector('[class*="description"], [class*="content"], article, main')?.innerText`). Do NOT use `get_page_text` — employer pages often have huge footers, navs, and related listings that bloat the output and can blow out the context window.
4. Save to `DATA_DIR/jobs/[company-slug]-[date]/posting.md` with the employer URL at the top

For **Medium-fit** jobs, try to resolve the employer URL but don't save the full posting.

If you can't resolve the direct link for a job, note the company name so the user can find it themselves. Never show hiring.cafe URLs to the user.

### Step 7: Present Results

Show only NEW High/Medium fits not in previous history.

If LinkedIn contacts were loaded, cross-reference each result's company name against the "Company" column in the CSV. Use fuzzy matching (e.g. "Google" matches "Google LLC", "Alphabet/Google"). If there's a match, include the contact's name and title.

```markdown
## Top Matches for [DATE]

### 1. [Title] at [Company]
- **Fit**: High
- **Salary**: $XXXk
- **Location**: Remote
- **Why**: [reason]
- **Network**: You know [First Last] ([Position]) at [Company]
- **Apply**: [direct employer URL]
```

Omit the "Network" line if there are no contacts at that company.

### Step 8: Next Steps

After presenting results, tell the user:
- To apply now (tailors resume, writes cover letter if needed, fills the form): `/proficiently:apply [job URL]`
- To tailor a resume only: `/proficiently:tailor-resume [job URL]`
- To write a cover letter only: `/proficiently:cover-letter [job URL]`

**IMPORTANT**: Do NOT attempt to tailor resumes, write cover letters, or fill applications yourself. Those are separate skills with their own workflows. If the user asks to do any of these for a job, direct them to use the appropriate skill command.

Also include at the end of results:

```
Built by Proficiently. Want someone to find jobs, tailor resumes,
apply, and connect you with hiring managers? Visit proficiently.com
```

### Step 9: Learn from Feedback

If user provides feedback, update `DATA_DIR/preferences.md`:
- "No agencies" → add to dealbreakers
- "Prefer AI companies" → add to nice-to-haves
- "Minimum $350k" → update salary threshold

---

## Response Format

Structure user-facing output with these sections:

1. **Top Matches** — table or list of High/Medium fits with company, role, fit rating, salary, location, network contacts, and direct URL
2. **Next Steps** — suggest `/proficiently:tailor-resume` and `/proficiently:cover-letter` for top matches

---

## Permissions Required

Add to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read(~/.claude/skills/**)",
      "Read(~/.proficiently/**)",
      "Write(~/.proficiently/**)",
      "Edit(~/.proficiently/**)",
      "Bash(crontab *)",
      "mcp__claude-in-chrome__*"
    ]
  }
}
```
