# Proficiently

A Claude Code plugin for AI-powered job searching, resume tailoring, and cover letter writing. Built by [Proficiently](https://proficiently.com).

> **Want someone to handle your entire job search?** Proficiently finds you jobs, tailors your resume and cover letters, applies on your behalf, and gets you in touch with hiring managers. Visit [proficiently.com](https://proficiently.com) to get started.

![proficiently-demo](https://github.com/user-attachments/assets/2f39a093-bf7b-4c43-a7b5-c3e6251440e4)

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [Setup](./skills/setup/) | `/proficiently:setup` | One-time onboarding: resume, preferences, LinkedIn contacts, and work history interview |
| [Job Search](./skills/job-search/) | `/proficiently:job-search` | Automated job search across hiring.cafe and LinkedIn with smart filtering and network matching |
| [Tailor Resume](./skills/tailor-resume/) | `/proficiently:tailor-resume` | Create tailored resumes for specific job postings |
| [Cover Letter](./skills/cover-letter/) | `/proficiently:cover-letter` | Write natural, persuasive cover letters |
| [Network Scan](./skills/network-scan/) | `/proficiently:network-scan` | Scan your contacts' companies for matching job openings |
| [Apply](./skills/apply/) | `/proficiently:apply` | Fill out job applications on Greenhouse, Lever, Workday, and LinkedIn Easy Apply |
| [Telegram Loop](./skills/jobsearch-telegram/) | `/proficiently:jobsearch-telegram` | Headless job search assistant via Telegram — apply, search, and check status by chat |

## How They Work Together

1. **`/proficiently:setup`** uploads your resume, configures preferences, imports LinkedIn contacts, and conducts a work history interview (one-time)
2. **`/proficiently:job-search`** finds jobs that match your preferences and resume, flags companies where you have connections
3. **`/proficiently:tailor-resume`** rewrites your resume for a specific job posting, saves the job posting and tailored resume together
4. **`/proficiently:cover-letter last`** writes a cover letter using the most recent job's posting and tailored resume
5. **`/proficiently:apply last`** fills out the application form on Greenhouse, Lever, Workday, or LinkedIn Easy Apply using your tailored resume and cover letter
6. **`/proficiently:network-scan`** scans your LinkedIn contacts' companies for matching openings (leverages your network for warm intros)
7. **`/loop 1m /proficiently:jobsearch-telegram`** runs the Telegram bot in the background — send a job URL or "search [keywords]" from your phone to trigger any of the above automatically

All skills share a `~/.proficiently/` directory for personal files. Each job application gets its own folder containing the posting, tailored resume, and cover letter.

## Installation

### Option A: Claude Cowork (desktop app)

1. Download [Claude Cowork](https://claude.com/product/cowork) if you haven't already
2. Download the plugin as a zip from GitHub: [Download ZIP](https://github.com/proficientlyjobs/proficiently-claude-skills/archive/refs/heads/main.zip)
3. In Cowork, go to **Plugins** (left sidebar) and click the **+** button
4. Select **Upload plugin**
5. Drag and drop the downloaded zip file, then click **Upload**
6. Run `/proficiently:setup` to get started

### Option B: Claude Code CLI

First, add the repository as a marketplace:

```bash
claude plugin marketplace add https://github.com/proficientlyjobs/proficiently-claude-skills.git
```

Then install the plugin:

```bash
claude plugin install proficiently@proficiently
```

Then run setup:

```
/proficiently:setup
```

### After installing

Setup will create `~/.proficiently/`, prompt you for your resume, configure your job preferences, optionally import your LinkedIn contacts, and conduct a work history interview.

You can also add your resume manually first:

```bash
mkdir -p ~/.proficiently/resume
cp /path/to/your/resume.pdf ~/.proficiently/resume/
```

## Prerequisites

- [Claude Cowork](https://claude.com/product/cowork) desktop app **or** [Claude Code CLI](https://claude.ai/code)
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension (for browser automation)
- Chrome browser running with the extension active

## File Structure

**Plugin (installed via marketplace):**
```
proficiently-claude-skills/
├── .claude-plugin/
│   └── plugin.json                     # Plugin manifest
├── shared/
│   ├── templates/
│   │   └── profile.md                  # Work history profile template
│   └── references/
│       ├── fit-scoring.md              # Canonical fit scoring criteria
│       ├── data-directory.md           # Data directory resolution algorithm
│       ├── prerequisites.md            # Prerequisites checking by skill
│       ├── browser-setup.md            # Browser automation setup sequence
│       ├── ats-patterns.md            # ATS navigation patterns (Greenhouse, Lever, Workday, LinkedIn)
│       └── priority-hierarchy.md       # Instruction priority hierarchy
├── skills/
│   ├── setup/
│   │   ├── SKILL.md
│   │   └── scripts/
│   ├── job-search/
│   │   ├── SKILL.md
│   │   ├── assets/templates/
│   │   └── scripts/
│   ├── tailor-resume/
│   │   ├── SKILL.md
│   │   └── scripts/
│   ├── cover-letter/
│   │   ├── SKILL.md
│   │   └── scripts/
│   ├── network-scan/
│   │   ├── SKILL.md
│   │   └── scripts/
│   ├── apply/
│   │   ├── SKILL.md
│   │   └── scripts/
│   └── jobsearch-telegram/
│       └── SKILL.md
└── README.md
```

**User data (created by `/proficiently:setup`, persists across plugin updates):**
```
~/.proficiently/
├── resume/                             # Your resume PDF/DOCX
├── profile.md                          # Work history from interview
├── preferences.md                      # Job matching rules
├── linkedin-contacts.csv               # LinkedIn connections (optional)
├── job-history.md                      # Running log from job-search
├── company-careers.json                # Cached careers page URLs
├── network-scan-history.md             # Running log from network-scan
├── application-data.md                # Reusable form field answers
└── jobs/                               # One folder per application
    ├── google-lead-gpm-2026-02-11/
    │   ├── posting.md                  # Saved job description
    │   ├── resume.md                   # Tailored resume
    │   ├── cover-letter.md             # Cover letter
    │   └── applied.md                  # Application log (date, ATS, status)
    └── ...
```

## Built by Proficiently

This plugin is free and open source. If you'd rather have someone handle the whole process for you — finding jobs, tailoring resumes, writing cover letters, submitting applications, and connecting you with hiring managers — visit [proficiently.com](https://proficiently.com).

## License

MIT
