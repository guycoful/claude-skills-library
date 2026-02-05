# Gmail Skill - Setup Guide

## Prerequisites

- Google account with Gmail
- Node.js (v16+)
- `clasp` CLI tool

## 1. Install Clasp

```bash
npm install -g @google/clasp
clasp login
```

Browser opens → authorize with your Google account.

## 2. Enable Apps Script API

Go to: https://script.google.com/home/usersettings
Toggle ON: "Google Apps Script API"

## 3. Create Apps Script Project

```bash
cd ~/.claude/skills/gmail/scripts

# Remove existing .clasp.json (has original project ID)
rm -f .clasp.json

# Create YOUR project
clasp create --title "Gmail API" --type webapp

# Push the code
clasp push
```

## 4. Set Your Secret Token

```bash
clasp open
```

In the Apps Script editor:
1. Find `setupToken()` function in Code.gs
2. Change `'YOUR_SECRET_TOKEN_HERE'` to a random string
3. Run > `setupToken`
4. Run > `testAuth` (grants Gmail permissions)

Generate a secure token:
```bash
openssl rand -hex 16
```

## 5. Deploy as Web App

In Apps Script UI: Deploy > New deployment > Web app:
- Execute as: **Me**
- Who has access: **Anyone**

Or via CLI:
```bash
clasp deploy --description "Gmail API v1"
clasp deployments  # Copy the URL
```

## 6. Configure Environment

Add to `~/.zshrc` or `~/.bashrc`:

```bash
export GMAIL_API_URL="https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec"
export GMAIL_API_TOKEN="your-secret-token"
```

Then: `source ~/.zshrc`

## 7. Test

```bash
# Test inbox
curl -sL "$GMAIL_API_URL?token=$GMAIL_API_TOKEN&action=inbox&maxResults=1"

# Test send (careful - this actually sends!)
curl -sL "$GMAIL_API_URL?token=$GMAIL_API_TOKEN&action=send&to=YOUR_EMAIL&subject=Test&body=Hello"
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| 401 error | Check token matches in Code.gs |
| Redirect loop | Use `-L` with curl |
| No emails returned | Check Gmail permissions granted |
| Script API not enabled | Enable at script.google.com/home/usersettings |

## Update Existing Deployment

```bash
clasp push
clasp deploy -i DEPLOYMENT_ID --description "v2"
```

## References

- [Code.js](scripts/Code.js) - The Apps Script code
