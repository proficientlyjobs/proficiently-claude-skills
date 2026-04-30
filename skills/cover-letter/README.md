# Cover Letter Skill for Coding Agents

Write natural, persuasive cover letters tailored to specific job postings. Works alongside the [tailor-resume](../tailor-resume/) and [job-search](../job-search/) skills.

## Features

- **Authentic voice** - sounds like a real professional, not AI
- **Achievement-focused** - connects 2-3 measurable results to the employer's specific needs
- **Strict honesty** - never fabricates or exaggerates any detail from the resume
- **Works with tailored resumes** - leverages existing match analysis when available

## Prerequisites

1. Claude Code, Codex, or another coding agent with this skill installed
2. Browser automation support. In Claude Code, use the [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension.
3. Resume and profile set up via `/proficiently:setup`

## Usage

Examples below use the Claude CLI. In Codex or another coding agent, invoke the same Proficiently skill by name with the same argument text.

### Write a cover letter for a job

```bash
claude "/cover-letter https://example.com/jobs/vp-growth"
```

### Use the most recent tailored resume

```bash
claude "/cover-letter last"
```

### General flow

```bash
claude "/cover-letter"
```

## How It Works

1. Reads your resume and work history profile
2. Gets the job posting (from URL or most recent tailored resume)
3. Identifies 2-3 achievements that directly address the employer's needs
4. Writes a 250-350 word cover letter in a natural, conversational tone
5. Saves to `~/.proficiently/jobs/[company-slug]/` for your review

## Tips

- Run `/tailor-resume` first, then `/cover-letter last` to get a cover letter that matches your tailored resume
- The work history interview makes cover letters significantly better since there's more material to draw from
- Every claim in the cover letter is verified against your actual resume and work history
