# OpenClaw Troubleshooting Guide
## The Complete Fix-It Handbook for OpenClaw Issues

---

## Introduction

This guide covers the most common OpenClaw problems and their solutions. Use this when your AI assistant isn't working as expected.

**Time Required:** 5-30 minutes depending on issue
**Difficulty:** Beginner to Intermediate

---

## Part 1: The First 60 Seconds (Quick Diagnosis)

Run these commands in order - one of them will reveal the problem:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

### What Good Output Looks Like

| Command | Expected Result |
|---------|-----------------|
| `openclaw status` | Configured channels, no auth errors |
| `openclaw gateway status` | Runtime: running, RPC probe: ok |
| `openclaw channels status --probe` | Channels show connected or ready |
| `openclaw logs --follow` | Steady activity, no repeating fatal errors |

---

## Part 2: Gateway Won't Start

### Symptom: "another gateway instance is already listening"

**Cause:** Port 18789 is already in use

**Fix:**
```bash
# Find and kill the existing process
lsof -i :18789
# Or restart the gateway
openclaw gateway restart
```

### Symptom: "Gateway start blocked: set gateway.mode=local"

**Cause:** Gateway mode is set to remote

**Fix in config:**
```json5
{
  gateway: {
    mode: "local"
  }
}
```

### Symptom: "refusing to bind gateway without auth"

**Cause:** Non-loopback bind without token/password

**Fix:**
```json5
{
  gateway: {
    bind: "127.0.0.1",
    auth: {
      mode: "token",
      token: "your-secure-token"
    }
  }
}
```

---

## Part 3: No Replies From the Bot

### Quick Checklist

1. Run: `openclaw status`
2. Check: Is gateway showing "Runtime: running"?
3. Check: Is your channel connected?
4. Check: Are you on the allowlist?

### Common Log Signatures

| Log Message | Meaning | Fix |
|-------------|---------|-----|
| `drop guild message (mention required)` | Mention gating blocked message | @mention the bot in Discord |
| `pairing request` | Sender unapproved | Run `openclaw pairing approve <code>` |
| `blocked` / `allowlist` | Sender filtered | Add sender to allowlist |
| `not_in_channel` | Missing channel permissions | Re-check bot has correct scopes |

### Fix Commands

```bash
# Check channel status
openclaw channels status --probe

# List pending pairings
openclaw pairing list --channel telegram
openclaw pairing list --channel whatsapp

# Approve a pairing
openclaw pairing approve --channel whatsapp <code>
```

---

## Part 4: Dashboard / Control UI Won't Connect

### Symptom: "unauthorized" or reconnecting

**Cause:** Wrong token or auth mode mismatch

**Fix:**
```bash
# Generate a new token
openclaw doctor --generate-gateway-token

# Get current token
openclaw config get gateway.auth.token
```

### Symptom: "gateway connect failed"

**Cause:** UI targeting wrong URL/port or unreachable gateway

**Fix:**
1. Check gateway is running: `openclaw gateway status`
2. Verify the URL matches (default: http://127.0.0.1:18789)
3. Check firewall settings

---

## Part 5: Model & Authentication Errors

### Symptom: "No credentials found for profile anthropic:default"

**Fix:**
```bash
# Option 1: Set API key in .env
echo "ANTHROPIC_API_KEY=sk-..." >> ~/.openclaw/.env

# Option 2: Use setup-token
claude setup-token
openclaw models auth paste-token --provider anthropic

# Option 3: Check env vars loaded
openclaw models status
```

### Symptom: "HTTP 429: rate_limit_error"

**Cause:** Anthropic quota exhausted

**Fix:**
- Wait for rate limit to reset
- Set a fallback model in config
- Upgrade your Anthropic plan

### Symptom: "Model is not allowed"

**Cause:** Model not in your allowlist

**Fix:**
```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5": {}
      }
    }
  }
}
```

---

## Part 6: Cron & Heartbeat Not Firing

### Checklist

```bash
# Check cron status
openclaw cron status
openclaw cron list

# Check recent runs
openclaw cron runs --id <jobId> --limit 20
```

### Common Causes

| Log Message | Meaning | Fix |
|-------------|---------|-----|
| `scheduler disabled` | Cron disabled | Enable in config |
| `heartbeat skipped: quiet-hours` | Outside active hours | Adjust heartbeat settings |
| `unknown accountId` | Delivery target doesn't exist | Check channel config |

### Fix Commands

```bash
# Enable cron
openclaw cron enable

# Test a cron job manually
openclaw cron run <jobId> --force

# Check heartbeat config
openclaw config get agents.defaults.heartbeat
```

---

## Part 7: Browser Tool Fails

### Symptom: "Failed to start Chrome CDP on port"

**Fix:**
```bash
# Check browser status
openclaw browser status

# Install browser manually
openclaw browser install
```

### Symptom: "Chrome extension relay is running, but no tab is connected"

**Fix:**
1. Click the OpenClaw Browser Relay toolbar icon on the tab
2. Confirm badge shows "ON"
3. Refresh the page and try again

### Symptom: "browser.executablePath not found"

**Fix in config:**
```json5
{
  browser: {
    executablePath: "/path/to/chromium"
  }
}
```

---

## Part 8: Node Tools Fail (Camera, Screen, Exec)

### Symptom: "NODE_BACKAVAILABLE"

**Fix:** Bring the node app to foreground on the paired device

### Symptom: "*_PERMISSION_REQUIRED"

**Fix:** Grant the required OS permission (camera, screen recording, etc.)

### Symptom: "SYSTEM_RUN_DENIED: approval required"

**Fix:** Approve the exec request on the node device

### Symptom: "SYSTEM_RUN_DENIED: allowlist miss"

**Fix:** Add command to the exec allowlist in config

---

## Part 9: Channel-Specific Issues

### WhatsApp

```bash
# Get group JID
openclaw logs --follow
# Send message in group, check logs for chatId ending in @g.us

# List groups
openclaw directory groups list --channel whatsapp
```

### Telegram

```bash
# Get your user ID
# DM @userinfobot to get your numeric ID

# Allowlist in config
{
  channels: {
    telegram: {
      allowFrom: ["123456789"]
    }
  }
}
```

### Discord

```bash
# Enable mention requirement
# Bot must be @mentioned to respond
```

---

## Part 10: Config Got Wiped

### Prevention

- Use `openclaw config set` for small changes
- Use `openclaw configure` for interactive edits
- NEVER use `config.apply` with partial objects

### Recovery

```bash
# Restore from backup
cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json

# Or re-run onboarding
openclaw onboard
```

---

## Emergency Reset

If all else fails:

```bash
# Full reset (keeps installation)
openclaw reset --scope full --yes

# Fresh start
openclaw onboard
```

---

## Get More Help

- **Docs:** https://docs.openclaw.ai
- **Discord:** https://discord.com/invite/clawd
- **GitHub Issues:** https://github.com/openclaw/openclaw/issues

---

*This guide was created to help you fix OpenClaw issues quickly. For the latest troubleshooting info, always check the official docs.*
