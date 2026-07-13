# LinkedIn DM Skill for Claude Code

Write short, tailored LinkedIn outreach messages to recruiters and hiring managers, optimized for open and reply rate. Works alongside the [cover-letter](../cover-letter/) and [tailor-resume](../tailor-resume/) skills, but for a different medium: a message that gets skimmed on a phone, not read like a letter.

## Features

- **Two variants every time** - a ≤300-character connection note and a ~500-character direct message, since LinkedIn behaves differently depending on connection status
- **One hook, not three** - picks the single strongest achievement for this specific recruiter/role instead of stacking credentials
- **Strict honesty** - never fabricates or exaggerates any detail from the resume
- **Character-counted** - connection notes are checked against LinkedIn's 300-character hard limit before being presented

## Prerequisites

1. [Claude Code CLI](https://claude.ai/code) installed
2. [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension installed
3. Resume and profile set up via `/proficiently:setup`

## Usage

### DM tailored to a recruiter's profile

```bash
claude "/linkedin-dm https://linkedin.com/in/recruiter-handle"
```

### DM tailored to a job posting

```bash
claude "/linkedin-dm https://linkedin.com/jobs/view/12345"
```

### Both (strongest personalization)

```bash
claude "/linkedin-dm https://linkedin.com/in/recruiter-handle https://linkedin.com/jobs/view/12345"
```

### Use the most recent job folder

```bash
claude "/linkedin-dm last"
```

## How It Works

1. Reads your resume and work history profile
2. Pulls the job posting and/or recruiter profile you point it at
3. Picks the single strongest angle for that specific person/role
4. Writes a connection request note (≤300 characters) and a direct message (~500 characters)
5. Saves both to `~/.proficiently/jobs/[company-slug]/linkedin-dm.md` for your review

## Tips

- Run `/tailor-resume` and `/cover-letter` first if you want the DM to stay consistent with what you've already sent
- Paste the recruiter's actual profile URL when you have it — a message with zero personalization performs like a template even if the words are different
- If they reply, that's the moment for `/cover-letter`, not before
