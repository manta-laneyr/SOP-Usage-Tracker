# SOP Usage Tracker Bot (GitHub Actions version)

Standalone weekly scan that replaces the Cowork scheduled task, which
can't currently load MCP connectors in its background execution
environment (open Anthropic platform bug). This runs on GitHub's own
cron instead, with direct API credentials to each service.

## What's different from the Cowork version

- **Asana matching**: uses the `typeahead` endpoint instead of full-text
  search, since `search_tasks` needs a paid Asana tier. Fuzzier match
  than the original — verify if exact parity matters.
- **Slack matching**: bot tokens can't do workspace-wide search
  (`search.messages` needs a legacy *user* token). This scans the
  channels listed in `SLACK_CHANNEL_IDS` instead — invite the bot to
  whichever channels are relevant.
- **Section 7 version-mismatch check**: placeholder only. I didn't have
  the original `sop-usage-tracker.html` source in this session to port
  the exact comparison logic, so this currently just flags docs that
  contain a "Section 7" heading for manual review. Share the original
  HTML if you want this wired up properly.

Everything else — folder ID, Slack DM target, and the active/used
recently/gone quiet/no signal thresholds — comes straight from the
project's key figures.

## One-time setup

### 1. Google Drive (read access to the SOP folder)
1. Go to console.cloud.google.com → create/select a project.
2. Enable the **Google Drive API**.
3. Create a **Service Account** (IAM & Admin → Service Accounts).
4. Create a JSON key for it and download it.
5. Open the SOP folder in Drive, share it with the service account's
   email (looks like `name@project.iam.gserviceaccount.com`), Viewer
   access.
6. Copy the *entire contents* of the JSON key file — you'll paste this
   as one GitHub secret.

### 2. Asana
1. Asana → your profile photo → My Settings → Apps → Manage Developer
   Apps → **Create New Personal Access Token**.
2. Also grab your workspace GID: visit
   `https://app.asana.com/api/1.0/workspaces` in a browser while
   logged in (or use the Asana API explorer) to find it.

### 3. Slack
1. Go to api.slack.com/apps → **Create New App** → From scratch →
   pick the `vansburg` workspace.
2. Under **OAuth & Permissions**, add Bot Token Scopes:
   `chat:write` (to send the DM) and `channels:history` /
   `groups:history` (to scan channels for mentions).
3. **Install to Workspace**, copy the **Bot User OAuth Token**
   (starts with `xoxb-`).
4. Invite the bot to whichever channels you want scanned for SOP
   mentions, and note their channel IDs for `SLACK_CHANNEL_IDS`.

### 4. Email (Gmail)
1. Turn on 2-Step Verification on the Gmail account if not already on.
2. Google Account → Security → **App Passwords** → generate one for
   "Mail".
3. Use that 16-character password as `EMAIL_APP_PASSWORD` (not your
   regular Gmail password).

### 5. Add GitHub Secrets
Repo → Settings → Secrets and variables → Actions → New repository
secret, one per line below:

| Secret name | Value |
|---|---|
| `GOOGLE_SERVICE_ACCOUNT_JSON` | full contents of the service account JSON key |
| `ASANA_TOKEN` | Asana Personal Access Token |
| `ASANA_WORKSPACE_GID` | your Asana workspace ID |
| `SLACK_BOT_TOKEN` | `xoxb-...` bot token |
| `SLACK_CHANNEL_IDS` | comma-separated channel IDs, e.g. `C0123,C0456` |
| `EMAIL_USER` | the Gmail address sending the summary |
| `EMAIL_APP_PASSWORD` | the 16-char app password |
| `EMAIL_TO` | where the summary should land |

### 6. Test it
Actions tab → "Weekly SOP Usage Scan" → **Run workflow** (this is the
`workflow_dispatch` trigger). Check the run logs, then check Slack and
your inbox. Once that works, the Sunday 16:00 UTC cron will fire on
its own — no more silent failures, since this never depends on
Cowork's connector loading.
