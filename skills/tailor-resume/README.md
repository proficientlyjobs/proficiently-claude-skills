# Resume Tailoring Skill for Coding Agents

Create tailored resumes that make you the obvious candidate for any specific job posting. Uses your resume and work history profile to craft compelling, targeted resumes.

## Features

- **Job posting analysis** - fetches and parses job details from any URL
- **Intelligent tailoring** - rearranges, rewrites, and emphasizes the right experience
- **Level-appropriate framing** - calibrates language and emphasis to match the role's seniority
- **Assumption tracking** - flags guesses when no work history profile exists

## Prerequisites

1. Claude Code, Codex, or another coding agent with this skill installed
2. Browser automation support. In Claude Code, use the [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension.
3. Resume and profile set up via `/proficiently:setup`

## Usage

Examples below use the Claude CLI. In Codex or another coding agent, invoke the same Proficiently skill by name with the same argument text.

### Tailor resume for a job

```bash
claude "/proficiently:tailor-resume https://example.com/jobs/vp-growth"
```

### General flow

```bash
claude "/proficiently:tailor-resume"
```

This will check prerequisites, then ask for a job URL.

## File Structure

**Plugin files:**
```
tailor-resume/
├── SKILL.md                          # Main skill definition
├── README.md                         # This file
└── scripts/
    └── tailor-resume.md              # Tailoring agent prompt
```

**User data (at `~/.proficiently/`):**
```
~/.proficiently/
├── resume/                           # Your resume PDF/DOCX
├── profile.md                        # Work history from interview
├── preferences.md                    # Job preferences (for context)
└── jobs/
    └── [company-slug]/
        ├── posting.md                # Saved job description
        └── resume.md                 # Tailored resume
```

## How It Works

1. **Checks prerequisites** - resume and work history profile must exist (via `/proficiently:setup`)
2. **Fetches the job posting** via browser automation
3. **Maps your experience** to the job's requirements
4. **Generates a tailored resume** with reordered bullets, rewritten descriptions, and a targeted summary
5. **Saves the output** for your review and iteration

## Tips

- The work history interview is the biggest unlock. A 15-minute conversation gives dramatically better tailored resumes.
- You can iterate on any generated resume - ask to adjust tone, emphasis, or specific bullets.
- Tailored resumes are saved with the company name and date, so you can track what you've sent where.
- The skill never fabricates experience - it reorganizes and reframes what's real.
