# OpenClaw Quick Fixes Checklist
## One-Page Reference for Common Issues

---

## 🚀 Run This First (Any Problem)

```bash
openclaw status
openclaw gateway status
openclaw doctor
```

---

## Problem: Bot Won't Reply

- [ ] `openclaw status` shows "Runtime: running"
- [ ] `openclaw channels status --probe` shows connected
- [ ] Check logs: `openclaw logs --follow`
- [ ] Are you on the allowlist? (check config)
- [ ] @mention the bot in Discord groups

**Log clues:**
- `mention required` → @mention bot
- `pairing request` → `openclaw pairing approve <code>`
- `blocked` → Check allowlist

---

## Problem: Gateway Won't Start

- [ ] Port in use? → `lsof -i :18789`
- [ ] Another instance running? → `openclaw gateway restart`
- [ ] Auth token set? → Check `gateway.auth.token` in config

**Error: "EADDRINUSE"**
```bash
openclaw gateway stop
openclaw gateway start
```

**Error: "refusing to bind"**
```json5
{ "gateway": { "auth": { "mode": "token", "token": "xxx" } } }
```

---

## Problem: Dashboard Won't Connect

- [ ] Gateway running? `openclaw gateway status`
- [ ] Right URL? http://127.0.0.1:18789
- [ ] Token correct? `openclaw doctor --generate-gateway-token`
- [ ] Try: `openclaw dashboard`

---

## Problem: Model/Auth Errors

**"No credentials found"**
```bash
# Check what's loaded
openclaw models status

# Add API key
echo "ANTHROPIC_API_KEY=sk-..." >> ~/.openclaw/.env

# Or use setup-token
claude setup-token
openclaw models auth paste-token --provider anthropic
```

**"HTTP 429 rate_limit_error"**
- Wait for reset
- Add fallback model
- Upgrade plan

**"Model is not allowed"**
- Add to `agents.defaults.models` in config

---

## Problem: Cron/Heartbeat Not Working

- [ ] Cron enabled? `openclaw cron status`
- [ ] Gateway running 24/7?
- [ ] Check logs for "scheduler disabled" or "quiet-hours"
- [ ] Test manually: `openclaw cron run <jobId> --force`

---

## Problem: Browser Tool Fails

- [ ] Chrome installed? `openclaw browser status`
- [ ] Extension attached? Click toolbar icon (badge ON)
- [ ] Headless mode? Check `browser.headless` config

---

## Problem: Node Tools Fail

- [ ] Node paired? `openclaw nodes status`
- [ ] Permissions granted? (camera, screen recording)
- [ ] App in foreground?
- [ ] Exec approved?

---

## Problem: WhatsApp Issues

- [ ] Get group JID: Check logs when sending to group
- [ ] Allowlist groups in config
- [ ] Pairing: `openclaw pairing approve whatsapp <code>`

---

## Problem: Telegram Issues

- [ ] Get user ID: DM @userinfobot
- [ ] Add to `channels.telegram.allowFrom`
- [ ] Check bot has correct permissions

---

## Problem: Config Wiped

**Don't let this happen:**
- Use `openclaw config set` NOT `config.apply` for changes
- Backup: `cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak`

**Recovery:**
```bash
openclaw reset
openclaw onboard
```

---

## Emergency Commands

```bash
# Full status
openclaw status --all

# Restart everything
openclaw gateway restart

# Generate new token
openclaw doctor --generate-gateway-token

# Fresh start
openclaw reset --scope full --yes
openclaw onboard
```

---

## Quick Config Fixes

**Add API key:**
```bash
echo "AN_KEY=sk-THROPIC_API..." >> ~/.openclaw/.env
```

**Allow all DMs (be careful):**
```json5
{
  channels: {
    whatsapp: { dmPolicy: "open" }
  }
}
```

**Set fallback model:**
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

---

## Where to Get Help

- **Docs:** docs.openclaw.ai
- **Discord:** discord.com/invite/clawd
- **GitHub:** github.com/openclaw/openclaw/issues

---

*Print this and keep it by your desk!*
