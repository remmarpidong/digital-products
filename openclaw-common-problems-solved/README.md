# OpenClaw Common Problems Solved
## 50+ Solutions to Issues You Face Daily

---

## Introduction

This guide provides direct solutions to the most common OpenClaw problems. No fluff - just the fix you need.

**How to use:** Find your problem, try the solution. If it doesn't work, try the next one.

---

## Section 1: Gateway Problems

### 1. Gateway Won't Start

**Error:** "Runtime: stopped" or "EADDRINUSE"

**Solutions (try in order):**

```bash
# 1. Simple restart
openclaw gateway restart

# 2. Check what's using the port
lsof -i :18789

# 3. Kill the process
kill -9 <PID>

# 4. Start fresh
openclaw gateway start
```

### 2. Gateway Starts But RPC Fails

**Error:** "RPC probe: failed"

```bash
# Check the actual error
openclaw gateway status

# Look for "Last gateway error"
# Common fixes:

# Restart with more info
openclaw gateway restart

# Check logs
openclaw logs --follow | tail -50
```

### 3. After Reboot, Gateway Won't Start

```bash
# Reinstall the service
openclaw gateway install

# Start it
openclaw gateway start

# Check status
openclaw gateway status
```

### 4. Wrong Config Loaded

**Error:** Config (cli) and Config (service) are different

```bash
# Force service to use current config
openclaw gateway install --force

# Or check which profile is active
echo $OPENCLAW_STATE_DIR
openclaw status --all
```

---

## Section 2: Connection Problems

### 5. Can't Access Dashboard

**Symptoms:** "Unauthorized" or page won't load

```bash
# Generate fresh token
openclaw doctor --generate-gateway-token

# Get current token
openclaw config get gateway.auth.token

# Or disable auth (not recommended for remote)
openclaw config set gateway.auth.mode "none"
openclaw gateway restart
```

### 6. Control UI Keeps Reconnecting

```bash
# Check gateway is running
openclaw gateway status

# Try refreshing with fresh token
openclaw dashboard
```

### 7. Remote Access Not Working

**Using Tailscale?**
```bash
# Make sure Tailscale is running
tailscale status

# Start with Tailscale serve
openclaw gateway --tailscale serve
```

**Using SSH Tunnel?**
```bash
# Create tunnel
ssh -N -L 18789:127.0.0.1:18789 user@host

# Then access http://127.0.0.1:18789
```

---

## Section 3: Model & Auth Problems

### 8. "No credentials found for profile anthropic:default"

```bash
# Option 1: Set API key
echo "ANTHROPIC_API_KEY=sk-..." >> ~/.openclaw/.env
openclaw gateway restart

# Option 2: Use setup-token
claude setup-token
openclaw models auth paste-token --provider anthropic

# Option 3: Check what's loaded
openclaw models status
```

### 9. Model Not Allowed

**Error:** "Model X is not allowed"

```bash
# Add to allowed models in config
openclaw config set agents.defaults.models.anthropic/claude-sonnet-4-5 "{}"

# Or remove the restriction entirely
# (remove agents.defaults.models from config)
```

### 10. Rate Limit Errors

**Error:** "HTTP 429: rate_limit_error"

```bash
# Wait 1-2 minutes

# Or add fallback model
openclaw config set agents.defaults.model.fallbacks '["openai/gpt-5-mini"]'
```

### 11. Model Switching Not Working

```bash
# Check available models
/model list

# Try a different model
/model sonnet

# Check model status
/model status
```

### 12. All Models Failed

```bash
# Check auth profiles
openclaw models status

# Look for auth errors in output
# Fix each provider:
# - Anthropic: API key or setup-token
# - OpenAI: API key
# - Google: API key
```

---

## Section 4: Channel Problems

### 13. WhatsApp - Not Receiving Messages

```bash
# Check channel status
openclaw channels status --probe

# Check logs
openclaw logs --follow | grep whatsapp

# Common fixes:
# 1. Re-scan QR code
# 2. Check phone number in allowlist
# 3. Check dmPolicy setting
```

### 14. WhatsApp - Can't Find Group JID

```bash
# Send a message in the group
# Then check logs
openclaw logs --follow | grep "chatId"

# Look for: XXXXXXXXX-XXXXXXXX@g.us
```

### 15. Telegram - Bot Won't Reply

```bash
# Check bot token
openclaw channels status

# Add yourself to allowlist
openclaw config set channels.telegram.allowFrom '["YOUR_USER_ID"]'

# Get your ID: DM @userinfobot
```

### 16. Telegram - Must @mention to Reply

**This is normal behavior for groups.**

For DMs, check:
```bash
openclaw config get channels.telegram.dmPolicy
# Should be "open" or "pairing"
```

### 17. Discord - Bot Ignores Messages

```bash
# Must @mention the bot in servers
# This is expected behavior

# Check channel config
openclaw channels status --probe

# Verify bot has correct permissions
```

### 18. Discord - Can't Send Messages

```bash
# Check bot permissions:
# - Send Messages
# - Read Message History
# - Embed Links

# Reinstall with correct scopes
openclaw channels reinstall discord
```

---

## Section 5: Message Problems

### 19. Messages Queue But Never Send

```bash
# Check delivery status
/deliver status

# Turn on delivery
/deliver on

# Check logs for errors
openclaw logs --follow
```

### 20. Long Response Times

```bash
# Check model being used
/model status

# Switch to faster model
/model haiku
/model gpt-mini

# Check for stuck processes
openclaw sessions list
```

### 21. Context Gets Truncated

```bash
# Compact the session
/compact

# Or start fresh
/new

# Increase context window (use bigger model)
```

---

## Section 6: Cron & Automation

### 22. Cron Jobs Not Running

```bash
# Check cron status
openclaw cron status

# Enable if disabled
openclaw cron enable

# Check jobs
openclaw cron list
```

### 23. Heartbeat Not Working

```bash
# Check heartbeat config
openclaw config get agents.defaults.heartbeat

# Check HEARTBEAT.md exists in workspace
ls -la ~/workspace/HEARTBEAT.md

# Outside quiet hours?
# Default quiet: 11 PM - 8 AM
```

### 24. Cron Job Failed

```bash
# Check recent runs
openclaw cron runs --id <jobId> --limit 10

# Run manually to test
openclaw cron run <jobId> --force
```

---

## Section 7: Browser & Tools

### 25. Browser Tool Fails

```bash
# Check browser status
openclaw browser status

# Install browser
openclaw browser install

# Check executable path
openclaw config get browser.executablePath
```

### 26. Chrome Extension Not Connecting

```bash
# Click the OpenClaw extension icon
# Badge should show "ON"

# If not, refresh the page
# Try again
```

### 27. Node Tools Not Working

```bash
# Check node status
openclaw nodes status

# Re-pair if needed
openclaw nodes pair

# Check permissions on node device
```

---

## Section 8: Config Problems

### 28. Config Got Wiped

**Prevention:**
- Never use `config.apply` with partial object
- Use `config set` for changes
- Backup regularly

**Recovery:**
```bash
# Restore from backup
cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json

# Or re-run onboarding
openclaw onboard
```

### 29. Can't Edit Config

```bash
# Check file permissions
ls -la ~/.openclaw/openclaw.json

# Fix if needed
chmod 644 ~/.openclaw/openclaw.json
```

### 30. Changes Not Applying

```bash
# Gateway must restart for most config changes
openclaw gateway restart

# Some changes hot-reload (check docs)
```

---

## Section 9: Session Problems

### 31. Context Too Long / Can't Send

```bash
# Compact session
/compact

# Or start new
/new
```

### 32. Session Stuck / Infinite Loop

```bash
# Interrupt the agent
stop
stop action
abort

# Or start fresh
/new
```

### 33. Want to Switch Topics

```bash
# Just start fresh
/new
```

---

## Section 10: Update Problems

### 34. Update Fails

```bash
# Check current version
openclaw --version

# Force update
openclaw update --force

# Or switch channels
openclaw update --channel stable
```

### 35. After Update, Nothing Works

```bash
# Check for breaking changes
openclaw logs --follow

# Rollback if needed
openclaw update --channel <previous-channel>

# Or reinstall
npm install -g openclaw@latest
```

---

## Section 11: Permission Problems

### 36. "Permission denied" Errors

```bash
# Check OpenClaw directory permissions
ls -la ~/.openclaw

# Fix ownership
sudo chown -R $(whoami) ~/.openclaw
```

### 37. Exec Commands Denied

```bash
# Check exec settings
openclaw config get agents.defaults.tools.system

# Add to allowlist if needed
# (requires config change)
```

---

## Section 12: Pairing Problems

### 38. Pairing Code Not Working

```bash
# Check pending pairings
openclaw pairing list --channel whatsapp
openclaw pairing list --channel telegram

# Approve
openclaw pairing approve --channel whatsapp <code>
```

### 39. Want to Remove Paired Device

```bash
# List pairings
openclaw pairing list

# Remove
openclaw pairing remove --channel whatsapp <device-id>
```

---

## Section 13: Memory Problems

### 40. Bot Forgets Everything

```bash
# Check MEMORY.md exists
ls -la ~/.openclaw/workspace/MEMORY.md

# Ask bot to remember
"Remember that I prefer afternoon meetings. Write to MEMORY.md"
```

### 41. Memory Search Not Working

```bash
# Check if memory search is enabled
openclaw config get memorySearch

# May need API key for embeddings
# Check: OpenAI, Gemini, or local provider
```

---

## Section 14: Multi-Agent Problems

### 42. Wrong Agent Responds

```bash
# Check agent bindings
openclaw agents list

# Switch agent
/agent <agent-name>

# Check routing config
```

---

## Section 15: Network Problems

### 43. Firewall Blocking Connection

```bash
# Check if port is open
nc -zv 127.0.0.1 18789

# Add rule if needed (Linux)
sudo ufw allow 18789
```

### 44. Can't Reach Remote Gateway

```bash
# Check network
ping <remote-ip>

# Check Tailscale
tailscale status

# Check SSH tunnel
ps aux | grep ssh
```

---

## Section 16: Installation Problems

### 45. Install Fails

```bash
# Check Node version
node --version
# Need 22+

# Try verbose mode
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

### 46. "Command not found" After Install

```bash
# Check npm global bin
npm config get prefix

# Add to PATH
export PATH="$PATH:$(npm config get prefix)/bin"

# Or source your profile
source ~/.bashrc
```

---

## Section 17: Logging Problems

### 47. Can't Find Logs

```bash
# Default location
ls /tmp/openclaw/

# Or use CLI
openclaw logs --follow
```

### 48. Logs Too Verbose

```bash
# Reduce log level
openclaw config set logging.level "warn"

# Or for even less
openclaw config set logging.level "error"
```

---

## Section 18: Performance Problems

### 49. Gateway Running Slow

```bash
# Check resource usage
top -u $(whoami)

# Check session count
openclaw sessions list

# Restart if needed
openclaw gateway restart
```

### 50. High Memory Usage

```bash
# Check what's using memory
openclaw sessions list

# Compact old sessions
# Or delete old sessions manually
rm ~/.openclaw/agents/*/sessions/*.jsonl
```

---

## Emergency Checklist

When everything fails:

```bash
# 1. Check gateway
openclaw gateway status

# 2. Check logs
openclaw logs --follow

# 3. Restart gateway
openclaw gateway restart

# 4. If still failing
openclaw doctor

# 5. Nuclear option
openclaw reset --scope full --yes
openclaw onboard
```

---

## Get Help

- **Docs:** docs.openclaw.ai
- **Discord:** discord.com/invite/clawd  
- **Issues:** github.com/openclaw/openclaw/issues

---

*Keep this guide handy - you'll need it!*
