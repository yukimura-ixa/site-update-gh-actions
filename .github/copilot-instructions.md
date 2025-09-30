# Copilot Instructions for site-update-gh-actions

## Project Overview

This repository contains a GitHub Actions workflow that monitors a target website for changes and sends notifications when updates are detected. It uses intelligent change detection to ignore trivial updates like session cookies and security tokens.

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── watch-site.yml    # Main workflow file
├── state/                     # Stores website state between runs
│   ├── url.txt               # Target URL being monitored
│   ├── last.html             # Latest HTML snapshot
│   ├── previous.id           # Previous change identifier
│   ├── current.id            # Current change identifier
│   └── diff.txt              # Diff output for detected changes
└── README.md                  # Project documentation
```

## Key Components

### GitHub Actions Workflow (`.github/workflows/watch-site.yml`)

The workflow runs on a schedule (4 times daily) and can be manually triggered. It:

1. **Fetches** the target website content
2. **Normalizes** content by removing session cookies and nonce values
3. **Detects** meaningful changes by comparing normalized content
4. **Generates** AI-powered summaries of changes using GitHub Models API
5. **Creates** GitHub issues with detailed change information
6. **Sends** Telegram notifications for immediate alerts
7. **Commits** state snapshots to the repository for historical reference

### Smart Change Detection

The workflow ignores trivial changes that don't represent actual content updates:

- **Set-Cookie headers**: `PHPSESSID`, `dwqa_anonymous`
- **JavaScript nonce values**: `pdpa_thailand.nonce` and similar CSRF tokens
- Content is normalized using `sed` to replace dynamic nonce values before comparison

### AI-Powered Summaries

Uses GitHub Models API (`gpt-4o-mini`) to generate human-readable summaries of detected changes.

## Development Guidelines

### When Modifying the Workflow

1. **Preserve existing functionality**: The workflow has carefully tuned change detection
2. **Test normalization**: Ensure trivial changes (cookies, nonces) are still ignored
3. **Validate notifications**: Check that issues and Telegram messages are only sent for meaningful changes
4. **Maintain state files**: Don't modify the `state/` directory structure without updating all references

### Key Configuration Points

- **Target URL**: Set in workflow env variable `URL` (line 21)
- **Schedule**: Defined in cron syntax (line 8) - currently 4 times daily in UTC
- **Normalization patterns**: 
  - Set-Cookie removal: line 60
  - Nonce replacement: lines 64, 93-94
- **AI Model**: `gpt-4o-mini` via GitHub Models API (line 162)

### Required Secrets

The workflow requires these GitHub repository secrets:
- `GITHUB_TOKEN` (automatically provided by GitHub Actions)
- `TELEGRAM_BOT_TOKEN` (optional, for Telegram notifications)
- `TELEGRAM_CHAT_ID` (optional, for Telegram notifications)

### Testing Changes

1. Use `workflow_dispatch` to manually trigger the workflow
2. Check workflow run logs in the Actions tab
3. Verify state files are updated correctly in the `state/` directory
4. Ensure issues are created only when `diff_lines > 0`

## Common Tasks

### Changing the Target Website

Edit line 21 in `.github/workflows/watch-site.yml`:
```yaml
URL: https://your-target-website.com
```

### Adjusting Monitoring Frequency

Edit the cron schedule on line 8:
```yaml
- cron: "0 1,2,8,9 * * *"  # Runs at 01:00, 02:00, 08:00, 09:00 UTC
```

### Adding New Normalization Rules

If the website has other dynamic elements that should be ignored:

1. Add sed patterns in the "Fetch page & compute change id" step (around line 64)
2. Apply the same patterns in the "Detect change" step (around lines 93-94)
3. Document the patterns in README.md under "Ignored Changes"

### Customizing AI Summaries

Edit the prompt template in the "Generate AI summary of changes" step (lines 151-155):
- Adjust focus areas (content vs. technical changes)
- Modify summary length (max_tokens parameter, line 167)
- Change temperature for more/less creative summaries (line 168)

## Code Style

- Use consistent bash scripting practices with `set -euo pipefail`
- Add descriptive comments for complex operations
- Follow existing YAML indentation (2 spaces)
- Keep step descriptions clear and concise

## Debugging Tips

1. **Check workflow logs**: Actions tab → Click on workflow run → Expand each step
2. **Inspect state files**: View files in `state/` directory after workflow runs
3. **Test normalization**: Run sed commands locally to verify pattern matching
4. **Validate diff output**: Review `state/diff.txt` to see what changes were detected

## Important Notes

- The workflow commits to the repository, so it needs `contents: write` permission
- Issues are only created when `diff_lines > 0` (meaningful content changes)
- First runs have no previous state to compare, so no issues are created
- State files should not be manually edited (they're managed by the workflow)
