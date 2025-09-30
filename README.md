# site-update-gh-actions

A GitHub Actions workflow to monitor website changes and send notifications.

## Features

- Monitors a target website on a scheduled basis (4 times daily)
- Detects meaningful content changes while ignoring trivial updates
- **AI-powered change summaries** using GitHub Models API
- Creates GitHub issues with detailed change information
- Sends Telegram notifications for immediate alerts
- Saves HTML snapshots for historical reference

## Smart Change Detection

The workflow intelligently ignores trivial changes that don't represent actual content updates:

### Ignored Changes
- **Set-Cookie headers**: Session IDs and cookies that change on every request
  - `PHPSESSID` - PHP session identifier
  - `dwqa_anonymous` - Anonymous user tracking cookie
- **JavaScript nonce values**: Security tokens that rotate regularly
  - `pdpa_thailand.nonce` - CSRF protection token in WordPress plugin

### How It Works

1. Downloads the website content and HTTP headers
2. Normalizes the content by:
   - Removing all `Set-Cookie` headers
   - Replacing all `"nonce":"..."` values with a placeholder
3. Computes a hash of the normalized content
4. Compares with the previous hash to detect changes
5. **Only triggers notifications when there are actual content differences (diff_lines > 0)**
6. Uses AI to generate a human-readable summary of detected changes

## Notifications

When a meaningful change is detected (with actual content differences):

- **GitHub Issue**: Created with detailed information including:
  - **AI-generated summary** of what changed on the website
  - Change identifier (ETag, Last-Modified, or content hash)
  - Commit SHA linking to the state snapshot
  - Number of changed lines
  - Normalized diff showing actual content changes
  - Explanation of what changes are ignored

- **Telegram Message**: Instant notification sent to configured chat

**Note**: Issues are NOT created for:
- First-time runs with no previous state to compare
- Changes with 0 diff lines (e.g., only normalized trivial changes)

## Usage

The workflow runs automatically on schedule. You can also trigger it manually from the GitHub Actions UI.

### Manual Trigger
1. Go to the Actions tab in your repository
2. Select "Watch website for updates"
3. Click "Run workflow"