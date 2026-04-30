# Job Search Skill for Coding Agents

An automated job search skill that uses browser automation to find and evaluate job listings from [hiring.cafe](https://hiring.cafe).

## Features

- **Automated daily search** via cron job
- **Smart filtering** based on your preferences (salary, location, dealbreakers)
- **Job history tracking** to avoid showing duplicates
- **Learning from feedback** - refine preferences over time
- **Browser automation** via the active agent's browser tools. In Claude Code, this is Claude in Chrome MCP.

## Prerequisites

1. Claude Code, Codex, or another coding agent with this skill installed
2. Browser automation support. In Claude Code, use the [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension.
3. Chrome or another supported browser running with the relevant automation extension/plugin active

## Installation

Install the Proficiently plugin through the Claude Code or Codex marketplace, then run setup.

```bash
claude "/proficiently:setup"
```

This will configure your resume, preferences, and work history profile.

## Usage

Examples below use the Claude CLI. In Codex or another coding agent, invoke the same Proficiently skill by name with the same argument text.

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

Just tell your coding agent what you want:
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

**Note**: Requires the browser and relevant automation extension/plugin to be active.

## Troubleshooting

### Permission prompts interrupting cron
For Claude Code, ensure all permissions are in `~/.claude/settings.json`. For Codex or other agents, grant equivalent filesystem, shell, web search, and browser automation access.

### Browser not responding
Make sure the browser and relevant automation extension/plugin are active.

### No jobs found
- Check that hiring.cafe is accessible
- Try different search terms
- Verify your matching rules aren't too restrictive

## Roadmap

### v0.1 - Current
- [x] Automated job search on hiring.cafe
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
- Support for additional job boards (LinkedIn, Indeed, etc.)
- Email digest of daily results
- Integration with ATS/application tracking
- Interview prep based on job description + work history

## Contributing

PRs welcome! Check the roadmap above for planned features, or open an issue to discuss new ideas.

## License

MIT
