# Job Search Skill for Claude Code

An automated job search skill that uses browser automation to find and evaluate job listings from [hiring.cafe](https://hiring.cafe) and [LinkedIn Jobs](https://www.linkedin.com/jobs/) (requires being signed into LinkedIn in Chrome).

## Features

- **Automated daily search** via cron job
- **Smart filtering** based on your preferences (salary, location, dealbreakers)
- **Job history tracking** to avoid showing duplicates
- **Learning from feedback** - refine preferences over time
- **Browser automation** via Claude in Chrome MCP

## Prerequisites

1. [Claude Code CLI](https://claude.ai/code) installed
2. [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension installed
3. Chrome browser running with the extension active

## Installation

### 1. Clone to your skills directory

```bash
git clone https://github.com/YOUR_USERNAME/job-search-skill.git ~/.claude/skills/job-search
```

### 2. Run setup

```bash
claude "/proficiently:setup"
```

This will configure your resume, preferences, and work history profile.

## Usage

### Daily search (manual)
```bash
claude "/job-search"
```

### Search with specific keywords
```bash
claude "/job-search AI infrastructure"
claude "/job-search remote startup"
```

### Run headless (for cron)
```bash
claude -p "/job-search"
```

## File Structure

**Plugin files:**
```
job-search/
├── SKILL.md                      # Main skill definition
├── README.md                     # This file
├── assets/
│   └── templates/                # Format templates (committed)
│       └── job-entry.md          # Format for history entries
└── scripts/
    └── evaluate-jobs.md          # Job evaluation subagent
```

**User data (at `~/.proficiently/`):**
```
~/.proficiently/
├── resume/                       # Your resume PDF/DOCX
├── preferences.md                # Job matching rules
├── job-history.md                # Log of all jobs found
└── jobs/                         # Per-job application folders
```

## Configuration

### Matching Rules (`~/.proficiently/preferences.md`)

Customize your job preferences:

```markdown
## Target Roles
- VP Growth
- Head of Growth
- Director of Marketing

## Must-Have Criteria
- Remote or hybrid OK
- Minimum $250k+ total comp

## Dealbreakers
- Marketing agencies
- Crypto/blockchain
- >25% travel required

## Nice-to-Have
- Series B+ startup
- AI/ML focus
- B2B SaaS
```

### Updating Preferences

Just tell Claude what you want:
- *"Add fintech to my nice-to-haves"*
- *"I don't want any roles requiring relocation"*
- *"Bump my minimum salary to $300k"*

The skill will update `~/.proficiently/preferences.md` automatically.

## Job History

All jobs found are logged to `~/.proficiently/job-history.md` with:
- Date and search terms
- Job details (title, company, location, salary)
- Fit score (High/Medium/Low/Skip)
- Notes explaining the rating

This prevents showing you the same jobs twice and creates a searchable archive.

## Cron Setup

To run daily at 9am:

```bash
# Add to crontab
(crontab -l 2>/dev/null; echo "0 9 * * * cd ~ && claude -p '/job-search' >> ~/.claude/skills/job-search/logs/cron.log 2>&1") | crontab -
```

**Note**: Requires Chrome to be running with Claude in Chrome extension active.

## Troubleshooting

### Permission prompts interrupting cron
Ensure all permissions are in `~/.claude/settings.json` (see Installation step 3).

### Browser not responding
Make sure Chrome is running and Claude in Chrome extension is active.

### No jobs found
- Check that hiring.cafe is accessible
- For LinkedIn results, make sure you're signed into LinkedIn in Chrome (a login wall causes LinkedIn to be skipped)
- Try different search terms
- Verify your matching rules aren't too restrictive

## Roadmap

### v0.1 - Current
- [x] Automated job search on hiring.cafe
- [x] LinkedIn Jobs as a second search source
- [x] Smart filtering (salary, location, dealbreakers)
- [x] Job history tracking
- [x] Daily cron automation
- [x] Self-configuring setup flow

### v0.2 - Work History Interview
- [ ] Interactive interview to capture detailed work history
- [ ] Deep-dive on accomplishments, metrics, and impact
- [ ] Store structured work history in `assets/work-history.md`
- [ ] Use as foundation for all application materials

### v0.3 - Tailored Applications
- [ ] Generate customized resume for each target job
- [ ] Generate tailored cover letters highlighting relevant experience
- [ ] Match work history accomplishments to job requirements
- [ ] Store generated materials in `assets/applications/`

### v0.4 - Proactive Company Research
- [ ] Research companies beyond job boards
- [ ] Identify target companies based on preferences (industry, stage, culture)
- [ ] Track companies not currently hiring but worth monitoring
- [ ] Alert when target companies post new roles

### Future Ideas
- Support for additional job boards (Indeed, etc.)
- Email digest of daily results
- Integration with ATS/application tracking
- Interview prep based on job description + work history

## Contributing

PRs welcome! Check the roadmap above for planned features, or open an issue to discuss new ideas.

## License

MIT
