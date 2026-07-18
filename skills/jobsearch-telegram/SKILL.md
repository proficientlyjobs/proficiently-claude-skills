---
name: jobsearch-telegram
description: Poll Telegram for job search messages — apply to jobs, search for roles, check status, all via chat
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch, mcp__claude-in-chrome__tabs_context_mcp, mcp__claude-in-chrome__tabs_create_mcp, mcp__claude-in-chrome__navigate, mcp__claude-in-chrome__read_page, mcp__claude-in-chrome__find, mcp__claude-in-chrome__form_input, mcp__claude-in-chrome__javascript_tool, mcp__claude-in-chrome__computer, mcp__claude-in-chrome__upload_image, mcp__claude-in-chrome__get_page_text, mcp__claude-in-chrome__read_console_messages
---

# Job Search Telegram Polling

Poll Telegram for incoming messages and route them to the appropriate Proficiently skill. Runs headlessly via `/loop 1m /proficiently:jobsearch-telegram`.

## First-Time Setup

Before this skill can run, the user must create a Telegram bot and configure it. If `DATA_DIR/telegram-config.md` does not exist, walk the user through setup:

### 1. Create a Telegram Bot

Tell the user:

> **Let's set up your Telegram bot.**
>
> 1. Open Telegram and search for **@BotFather**
> 2. Send `/newbot`
> 3. Choose a name (e.g., "My Job Search Assistant")
> 4. Choose a username (must end in `bot`, e.g., `my_jobsearch_bot`)
> 5. BotFather will give you a **bot token** — copy it and paste it here
>
> Then send your bot a message (anything) so I can find your chat ID.

### 2. Get the Chat ID

Once the user provides the bot token, fetch their chat ID:

```bash
curl -s "https://api.telegram.org/bot{TOKEN}/getUpdates"
```

Extract `message.chat.id` from the first result. If no results, remind the user to send a message to the bot first, then retry.

### 3. Save Config

Write `DATA_DIR/telegram-config.md`:

```markdown
# Telegram Config

- Bot token: {TOKEN}
- Chat ID: {CHAT_ID}
- Bot username: @{USERNAME}
```

### 4. Verify

Send a test message:

```bash
curl -s -X POST "https://api.telegram.org/bot{TOKEN}/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{"chat_id": "{CHAT_ID}", "text": "👋 Job search bot connected! Send me a job URL to apply, or say \"search\" to find jobs."}'
```

If successful, tell the user setup is complete and they can start the loop with `/loop 1m /proficiently:jobsearch-telegram`.

---

## Config & State Files

Resolve the data directory using `shared/references/data-directory.md`.

**Config** — `DATA_DIR/telegram-config.md` (created during setup, contains bot token + chat ID). **Never commit this file to git.** Read this first on every poll cycle to get credentials.

**State** — `DATA_DIR/telegram-state.md` (tracks polling position). Create if missing:

```markdown
# Telegram State

## Polling
- last_update_id: 0

## Pending Confirmations
<!-- Format: [msg_id: X] type/stage — description — waiting since DATE
     For apply confirmations, also store: job_url, form_url, field_mapping (JSON) -->

(none)

## Recent Actions
<!-- Last 20 actions taken -->
```

---

## Workflow

### Step 1: Load Context

1. Read `DATA_DIR/telegram-config.md` — if missing, run First-Time Setup above and stop
2. Read `DATA_DIR/telegram-state.md` — if missing, create from template above
3. Read these if they exist: `DATA_DIR/job-history.md`, `DATA_DIR/application-data.md`, `DATA_DIR/preferences.md`

### Step 2: Poll for Messages

```bash
curl -s "https://api.telegram.org/bot{TOKEN}/getUpdates?offset={LAST_UPDATE_ID+1}&timeout=5"
```

If no new messages → exit silently. Do not log, do not send anything.

### Step 3: Classify Each Message

Parse each message and classify:

| Message Type | Detection | Route |
|---|---|---|
| Job URL | Contains `greenhouse.io`, `lever.co`, `myworkdayjobs.com`, `linkedin.com/jobs`, `ashbyhq.com`, or other job board URL | Step 4a: Apply |
| "apply last" / "apply" | Text matches `apply` (with optional `last`/`current`) | Step 4a: Apply |
| "search for ..." | Text starts with `search`, `find`, `look for` | Step 4b: Search |
| "tailor resume for ..." | Text mentions `tailor`/`resume` + context | Step 4c: Tailor |
| "status" / "what's open" | Text asks about application status | Step 4d: Status |
| "help" | Text is exactly `help` or `?` | Step 4e: Help |
| Confirmation reply | **Threaded reply** to a pending confirmation message, OR standalone confirm word (`yes`/`y`/`go`/`no`/`cancel`) when pending confirmations exist | Step 5: Confirm |
| Plain text | Anything else | Step 6: Note |

### Step 4a: Handle Job URL / Apply Request

1. Extract the URL or resolve "last"/"current"
2. Check if a job folder already exists in `DATA_DIR/jobs/` for this URL
3. Send acknowledgment to Telegram:
   ```
   🎯 Got it — applying to [URL or "most recent job"].
   I'll scan the form, tailor your resume, and propose answers. Stand by...
   ```
4. **Execute the apply workflow** from `skills/apply/SKILL.md`:
   - Follow Steps 0-6 (prerequisites → navigate → scout → generate materials → scan fields → propose answers)
   - Instead of using AskUserQuestion for approval, **send the Step 6 proposal summary to Telegram** and add to Pending Confirmations
   - Store in the pending confirmation: `job_url`, `form_url` (the direct ATS form URL navigated to), and `field_mapping` (the full approved field→value JSON)
   - Wait for user confirmation via Telegram (will arrive as a reply in a future poll cycle)

5. When field-approval confirmation arrives (Step 5), re-navigate to `form_url`, fill all fields, then send a **second confirmation** (submit approval) with a screenshot description and ask: `"Everything looks good — submit?"`
   - Store this as a new pending confirmation with `stage: "submit-approval"`

6. When submit-approval arrives, click Submit, then log the application (Step 9 of the apply skill).

**Sending the proposal:** Use the send message helper (Step 8) with the full field summary. Keep it under 4000 chars. If longer, split into: (1) auto-fill fields, (2) proposed answers, (3) needs input.

**Two-phase confirmation flow:**
- Phase 1 (`stage: field-approval`): User approves the field→value mapping
- Phase 2 (`stage: submit-approval`): User approves the final form before clicking Submit
- Never skip phase 2 — submitting a job application is irreversible

### Step 4b: Handle Search Request

1. Extract search keywords from the message
2. Send acknowledgment: `🔍 Searching for: [keywords]...`
3. Execute the job-search workflow from `skills/job-search/SKILL.md`
4. Send results summary to Telegram:
   ```
   🔍 Found X matches for "[keywords]":

   1. [Role] at [Company] — [fit score]
      [URL]
   2. ...

   Reply with a number to apply, or "apply 1" / "apply 3" etc.
   ```
5. Add to Pending Confirmations with the job list so replies can be matched

### Step 4c: Handle Tailor Request

1. Extract job reference (URL, "last", or job name)
2. Send acknowledgment: `📝 Tailoring resume for [job]...`
3. Execute the tailor-resume workflow from `skills/tailor-resume/SKILL.md`
4. Send result to Telegram with key changes made
5. Note the file path where the tailored resume was saved

### Step 4d: Handle Status Query

Compile from `DATA_DIR/job-history.md` and `DATA_DIR/jobs/*/applied.md`:

```
📋 Job Search Status

Applied (X):
- [Role] at [Company] — [date] — [status]
- ...

Saved but not applied (Y):
- [Role] at [Company] — [date saved]
- ...

Pending your confirmation:
- [any pending apply proposals]
```

### Step 4e: Handle Help Request

Send:

```
👋 Here's what you can do:

<b>Apply</b>
• Send a job URL → I'll apply for you
• "apply last" → continue with the most recent job

<b>Search</b>
• "search [keywords]" → find matching jobs
• "find AI product jobs" → same thing

<b>Resume</b>
• "tailor resume for [job URL or name]"

<b>Status</b>
• "status" → see all applications and what's pending

<b>Other</b>
• "help" → this message
• Any other text is saved as a note
```

### Step 5: Handle Confirmation Reply

A confirmation reply is either:
- A **threaded reply** (Telegram's native reply feature): match via `reply_to_message.message_id` to a pending confirmation
- A **standalone message** containing only a confirm/reject word (`yes`, `y`, `go`, `send it`, 👍, `no`, `skip`, `cancel`, ❌) when pending confirmations exist

**Disambiguation when standalone:**
- If exactly one pending confirmation exists → apply it to that confirmation
- If multiple pending confirmations exist → respond with a numbered list of what's pending and ask which one they mean:
  ```
  You have X things waiting. Which one?
  1. [description of pending 1]
  2. [description of pending 2]
  Reply with a number.
  ```

**Processing:**
1. Look up the pending confirmation in `telegram-state.md`
2. Parse the user's reply:
   - "yes" / "y" / "go" / "send it" / 👍 → approve
   - "no" / "skip" / "cancel" / ❌ → reject
   - "yes but [changes]" → approve with modifications
   - A number (e.g., "2") → select that option from a list
3. Execute the approved action:
   - `stage: field-approval` → re-navigate to `form_url`, fill fields using `field_mapping`, then send submit-approval prompt
   - `stage: submit-approval` → click Submit, log application
   - Search result selection → start apply workflow for selected job
4. Send confirmation of what was done
5. Remove from Pending Confirmations, add to Recent Actions

### Step 6: Handle Plain Text Note

1. Log to `DATA_DIR/telegram-inbox.md` with timestamp
2. If it looks like a company name or job title, suggest: `"Want me to search for [text] jobs?"`
3. Otherwise confirm: `"Noted 👍"`

### Step 7: Update State

After processing all messages:
1. Update `last_update_id` in `DATA_DIR/telegram-state.md`
2. Update Pending Confirmations (add new, remove resolved). For each apply confirmation, include:
   ```
   [msg_id: X] apply/field-approval — [Role] at [Company] — waiting since DATE
   job_url: https://...
   form_url: https://...
   field_mapping: {"First Name": "...", "Email": "...", ...}
   ```
3. Update Recent Actions (keep last 20, newest first)

### Step 8: Sending Messages

Read credentials from `DATA_DIR/telegram-config.md`, then send via curl:

```bash
curl -s -X POST "https://api.telegram.org/bot{TOKEN}/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{"chat_id": "CHAT_ID", "text": "MESSAGE", "parse_mode": "HTML"}'
```

For replies to specific messages, add `"reply_to_message_id": MSG_ID`.

**Formatting rules:**
- Use HTML: `<b>bold</b>`, `<i>italic</i>`, `<code>code</code>`
- Keep messages under 4000 chars (Telegram limit is 4096)
- Be concise — user reads on mobile
- Use line breaks for readability, not walls of text

To capture the sent message's `message_id` (needed for tracking confirmations):
```bash
# Parse from response JSON
jq -r '.result.message_id'
```

---

## Key Rules

- **HEADLESS.** Never use AskUserQuestion. All interaction happens through Telegram.
- **Acknowledge fast.** Send a quick reply before starting long operations (apply, search, tailor).
- **Suggest before acting.** For apply: always send the proposal and wait for Telegram confirmation before filling fields.
- **Never skip submit confirmation.** Submitting an application is irreversible — always require explicit approval.
- **Exit silently** if no new messages. Don't log empty polls.
- **Concise messages.** Mobile-first. No walls of text.
- **Track everything** in telegram-state.md so confirmations persist across poll cycles.
- **No secrets in git.** Credentials live only in `DATA_DIR/telegram-config.md`.
- **Log costs every cycle.** See Cost Tracking below.

## Cost Tracking

After every poll cycle that does actual work (not silent exits), you MUST:

1. **Count your token usage** for this cycle. At the end of your response, estimate:
   - **Input tokens**: approximate total from all files read + tool results received
   - **Output tokens**: approximate total from all text + tool calls you generated
   - Use these rates: input = $3/M tokens, output = $15/M tokens (cache reads = $1.875/M)

2. **Append to `DATA_DIR/telegram-cost-log.csv`** (create with header if missing):

   ```csv
   timestamp,action,input_tokens,output_tokens,estimated_cost_usd
   ```

   Example row:
   ```csv
   2026-03-11T14:30:00Z,apply-proposal,12000,3500,$0.09
   ```

3. **Include a cost footer in every Telegram reply**:
   ```
   ---
   📊 ~12K in / ~3.5K out · ~$0.09
   ```

4. **On "status" queries**, include a cost summary section:
   ```
   💰 Cost this session: $X.XX (Y interactions)
   💰 Cost all-time: $X.XX (Z interactions)
   ```
   Compute from the CSV log.

## Permissions Required

Add to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(curl:*)",
      "Bash(jq:*)",
      "Read(~/.proficiently/**)",
      "Write(~/.proficiently/**)",
      "Edit(~/.proficiently/**)",
      "Read(~/.claude/skills/**)",
      "mcp__claude-in-chrome__*"
    ]
  }
}
```
