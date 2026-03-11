# OpenClaw Daily Handbook
## Your Day-to-Day Guide for Running OpenClaw

---

## Introduction

This handbook covers daily tasks, common workflows, and solutions to problems you'll encounter when using OpenClaw as your personal AI assistant.

**Who this is for:** OpenClaw users who want to get the most out of their AI assistant
**Time to read:** 15-20 minutes

---

## Chapter 1: Daily Commands You Need

### Morning Health Check

Start your day with these commands:

```bash
# Quick status
openclaw status

# Full report
openclaw status --all

# Gateway health
openclaw gateway status
```

### Check Running Tasks

```bash
# See what the agent is doing
openclaw sessions list

# Check cron jobs
openclaw cron status
openclaw cron list
```

---

## Chapter 2: Common Daily Tasks

### Sending Messages

```bash
# Send via WhatsApp
openclaw message send --channel whatsapp --target +15551234567 --message "Hey!"

# Send via Telegram
openclaw message send --channel telegram --target 123456789 --message "Hello"

# Send to Discord channel
openclaw message send --channel discord --target "#general" --message "Update time"
```

### Starting New Conversations

```bash
# New session (fresh context)
/new

# Or
/reset
```

These commands start a fresh session - useful when switching topics.

### Switching Models

```bash
# Switch to a different model
/model sonnet
/model opus
/model gpt

# List available models
/model list
/model status
```

---

## Chapter 3: Daily Automation

### Setting Up Cron Jobs

```bash
# Create a daily summary job
openclaw cron add --name "morning-brief" \
  --schedule "0 9 * * *" \
  --message "Give me a morning summary of my emails and calendar" \
  --delivery announce
```

### Heartbeat Setup

Heartbeats run every 30 minutes by default. Configure in your workspace:

```bash
# Edit HEARTBEAT.md in your workspace
# Add tasks you want checked regularly
```

Example HEARTBEAT.md:
```markdown
# HEARTBEAT.md
- Check calendar for upcoming meetings
- Check for urgent emails
- Report weather if user might go out
```

---

## Chapter 4: Troubleshooting Daily Issues

### Problem: Gateway Won't Start

**Symptoms:** 
- `openclaw gateway status` shows "Runtime: stopped"
- Can't connect to dashboard

**Solutions:**

```bash
# Try restarting
openclaw gateway restart

# Check logs
openclaw logs --follow

# If port in use
lsof -i :18789
kill -9 <PID>
```

### Problem: No Replies

**Symptoms:** Messages sent but no response

**Solutions:**

1. Check channel status:
```bash
openclaw channels status --probe
```

2. Check logs:
```bash
openclaw logs --follow
```

3. Common causes:
- Not on allowlist → Add to config
- Mention required → @mention bot (Discord)
- Pairing pending → `openclaw pairing approve <code>`

### Problem: Model Errors

**"No credentials found"**
```bash
# Check what's loaded
openclaw models status

# Add API key
echo "ANTHROPIC_API_KEY=sk-..." >> ~/.openclaw/.env
openclaw gateway restart
```

**"Rate limited"**
- Wait for cooldown
- Add fallback model:
```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5-mini"]
      }
    }
  }
}
```

### Problem: Messages Not Delivered

Check delivery settings:
```bash
/deliver on
/deliver status
```

### Problem: Context Too Long

```bash
# Compact session
/compact

# Start fresh
/new
```

---

## Chapter 5: Multi-Channel Management

### Managing Multiple Chats

```bash
# List active sessions
openclaw sessions list

# Switch sessions
/focus <session-name>

# Unbind from thread/channel
/unfocus
```

### Channel-Specific Tips

**WhatsApp:**
- Groups need JID added to config
- Use `openclaw directory groups list --channel whatsapp`

**Telegram:**
- Bot must be @mentioned in groups
- DM works immediately after pairing

**Discord:**
- Must @mention to get response
- Can use threads for separate conversations

---

## Chapter 6: Productivity Workflows

### Morning Briefing Workflow

1. Set up cron job for 8-9 AM
2. Include email check, calendar, weather
3. Get summary in your chat

### Research Workflow

1. Use web search: `search for [topic]`
2. Use web fetch: `read [URL]`
3. Summarize findings

### Task Management

1. Create todo list in workspace
2. Use cron to remind
3. Track progress in memory files

---

## Chapter 7: Configuration Essentials

### Essential Config Settings

```json5
{
  // Always-on gateway
  gateway: {
    mode: "local",
    bind: "127.0.0.1",
    port: 18789
  },
  
  // Your primary channel
  channels: {
    whatsapp: {
      allowFrom: ["+15551234567"]
    }
  },
  
  // Model settings
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5"
      }
    }
  }
}
```

### Updating Config Safely

```bash
# DON'T use config.apply with partial objects!

# DO use config set for small changes
openclaw config set agents.defaults.model.primary "anthropic/claude-opus-4-6"

# DO use configure for interactive editing
openclaw configure --section agents
```

---

## Chapter 8: Memory & Continuity

### How Memory Works

- **Session:** Current conversation (can get long)
- **Daily notes:** `memory/YYYY-MM-DD.md`
- **Long-term:** `MEMORY.md`

### Making Memory Stick

Ask the bot to write important things to MEMORY.md:

```
Remember that I prefer morning meetings. Write this to MEMORY.md
```

### Checking Memory

```bash
# Search memory
# Just ask the bot!

"What's in my MEMORY.md about my preferences?"
```

---

## Chapter 9: Remote Access

### Accessing Your Gateway Remotely

**Option 1: SSH Tunnel**
```bash
ssh -N -L 18789:127.0.0.1:18789 user@vps
# Then open http://127.0.0.1:18789
```

**Option 2: Tailscale**
```bash
openclaw gateway --tailscale serve
# Access at https://your-vps.tailnet.ts.net
```

**Option 3: Cloudflare Tunnel**
```bash
openclaw gateway --cloudflare tunnel
```

---

## Chapter 10: Maintenance Tasks

### Weekly Maintenance

```bash
# Check disk usage
df -h ~/.openclaw

# Clean old logs (optional)
rm /tmp/openclaw/openclaw-2025-*.log

# Backup workspace
cp -r ~/.openclaw/workspace ~/backup/workspace-$(date +%Y%m%d)
```

### Monthly Maintenance

```bash
# Update OpenClaw
openclaw update

# Review cron jobs
openclaw cron list

# Check session sizes
du -sh ~/.openclaw/agents/*/sessions
```

### Updating OpenClaw

```bash
# Check for updates
openclaw update status

# Update to latest
openclaw update

# Or specific channel
openclaw update --channel stable
openclaw update --channel beta
```

---

## Quick Reference Card

### Must-Know Commands

| Command | Purpose |
|---------|---------|
| `openclaw status` | Quick health check |
| `openclaw gateway restart` | Restart gateway |
| `/new` | Fresh session |
| `/model [name]` | Switch model |
| `/compact` | Summarize context |
| `/deliver on/off` | Toggle delivery |

### Emergency Commands

```bash
# Gateway completely stuck
openclaw gateway stop
openclaw gateway start

# Full reset (keeps installation)
openclaw reset --scope full --yes

# Generate new token
openclaw doctor --generate-gateway-token
```

---

## Get More Help

- **Documentation:** docs.openclaw.ai
- **Discord:** discord.com/invite/clawd
- **GitHub:** github.com/openclaw/openclaw

---

*This handbook helps you get the most out of OpenClaw every day. Check for updates regularly!*
