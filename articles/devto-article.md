# OpenClaw Getting Started Guide: Build Your Personal AI Assistant in 5 Minutes

A step-by-step guide to installing and configuring OpenClaw from scratch. Connect WhatsApp, Telegram, Discord, and more. Perfect for beginners.

---

## Introduction: Why You Need OpenClaw

What if you could have an AI assistant that runs entirely on your own hardware, with complete control over your data, and works on all the chat apps you already use?

That's OpenClaw.

### What is OpenClaw?

OpenClaw is a **self-hosted gateway** that bridges your favorite chat applications (WhatsApp, Telegram, Discord, WeChat, iMessage, etc.) with AI models (OpenAI GPT, Anthropic Claude, Google Gemini, local Ollama, and more).

In plain English: **You send a message from your chat app → OpenClaw calls an AI → the AI's reply comes back to your chat app.** All on your infrastructure, your rules.

### Why Choose OpenClaw Over ChatGPT or Claude?

| Feature | OpenClaw | ChatGPT/Claude (Cloud) |
|---------|----------|----------------------|
| **Data Privacy** | ✅ 100% in your control | ❌ Third-party servers |
| **Multi-platform** | ✅ 20+ apps simultaneously | ❌ Single official app |
| **Cost Control** | ✅ Pay only API costs (or $0 local) | ❌ Monthly subscription |
| **Customization** | ✅ Full control over prompts, models | ❌ Limited |
| **No Vendor Lock-in** | ✅ Switch AI models anytime | ❌ Tied to one provider |

If you value privacy, hate monthly subscriptions, or want an AI that works across all your chat apps, OpenClaw is for you.

---

## Quick Start: 5-Minute Installation

### Prerequisites

- A computer (Linux/macOS/Windows) with at least 4GB RAM
- Basic terminal skills
- (Optional) An AI API key from OpenAI, Anthropic, or Ollama installed locally

### Installation

Open your terminal and run:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

The installer will:
- Detect your OS
- Install Node.js if missing
- Install the OpenClaw CLI
- Launch the onboarding wizard

### Onboarding Wizard

After installation, run:

```bash
openclaw onboard --install-daemon
```

You'll be prompted to:

1. **Choose your AI provider**:
   - OpenAI (requires API key)
   - Anthropic (requires API key)
   - Local (Ollama) - recommended for privacy and zero API costs
   - Google Gemini
   - Custom provider

2. **Enter your API key** (if using cloud provider)

3. **Configure your assistant identity**:
   - Name (default: "OpenClaw Assistant")
   - System prompt (customize behavior)

4. **Choose initial chat platform**:
   - You can add more platforms later

5. **Start the gateway** (runs in background)

That's it! You now have a personal AI assistant running on your own hardware.

---

## Connecting Your Chat Platforms

OpenClaw supports 20+ platforms. Here's how to connect the most popular ones.

### WhatsApp

1. Install the WhatsApp plugin:
```bash
openclaw plugins install whatsapp
```

2. Scan the QR code from the OpenClaw logs:
```bash
openclaw urls
# Look for WhatsApp QR code URL
```

3. Open the URL on your phone, scan with WhatsApp app

4. Start chatting!

### Telegram

1. Create a bot via [@BotFather](https://t.me/botfather):
   - Send `/newbot`
   - Choose a name and username
   - Copy the bot token

2. Configure OpenClaw:
```bash
openclaw config edit
```

Add to `plugins.telegram`:
```json5
{
  "plugins": {
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

3. Restart:
```bash
openclaw restart
```

4. Search your bot in Telegram and start chatting.

### Discord

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a New Application → Bot → Copy Token
3. Enable **Message Content Intent** (required to read messages)
4. Invite bot to your server with `Send Messages` and `Read Message History` permissions
5. Add token to `plugins.discord` in config
6. Restart OpenClaw

### Feishu (飞书)

Feishu requires a few more steps due to OAuth:

1. Create an app at [Feishu Developer Console](https://open.feishu.cn/app)
2. Add permissions: `im:message`, `im:chat`
3. Enable event subscription for `im.message.receive_v1`
4. Get App ID and App Secret
5. Configure in OpenClaw:
```json5
{
  "plugins": {
    "feishu": {
      "appId": "YOUR_APP_ID",
      "appSecret": "YOUR_APP_SECRET"
    }
  }
}
```
6. Get the webhook URL from `openclaw urls` and set in Feishu event subscription
7. Install the bot to your chat (or org-wide)

Detailed Feishu setup: [docs.openclaw.ai/plugins/feishu](https://docs.openclaw.ai/plugins/feishu)

---

## Advanced Configuration

### Multi-Model Support

You can configure multiple AI models with fallback:

```json5
{
  "models": {
    "default": {
      "provider": "openai",
      "model": "gpt-4o",
      "apiKey": "sk-xxx"
    },
    "fallback": {
      "provider": "ollama",
      "model": "llama3.2:3b",
      "baseUrl": "http://localhost:11434"
    }
  }
}
```

If the default model fails (rate limit, network issue), OpenClaw automatically switches to fallback.

### Channel-Specific Behavior

Customize assistant behavior per platform:

```json5
{
  "channels": {
    "telegram": {
      "agent": {
        "name": "TG Bot",
        "systemPrompt": "Be concise. Max 3 sentences."
      }
    },
    "discord": {
      "agent": {
        "name": "Discord Helper",
        "systemPrompt": "Be friendly, use emojis."
      }
    }
  }
}
```

### Caching

Enable response caching to reduce API costs:

```json5
{
  "cache": {
    "enabled": true,
    "ttl": 3600,
    "maxSize": 1000
  }
}
```

---

## Common Issues & Troubleshooting

### "Port 8080 already in use"

Change the port in config:
```json5
{
  "gateway": { "port": 8081 }
}
```

### Messages not arriving

1. Check gateway status:
```bash
openclaw status
```

2. View logs:
```bash
openclaw logs --follow
```

3. Ensure bot is online and properly configured

### API key invalid

Double-check:
- No extra spaces
- Correct format (`sk-` for OpenAI)
- Not expired

### SSL/HTTPS for production

For production deployments with custom domain:
```bash
# Use nginx or Caddy as reverse proxy with SSL
# See: https://docs.openclaw.ai/deployment/production
```

---

## Production Deployment (Optional)

For 24/7 operation and custom domain:

### Systemd Service (Linux)

Create `/etc/systemd/system/openclaw.service`:

```ini
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=yourname
WorkingDirectory=/home/yourname/.openclaw
ExecStart=/usr/local/bin/openclaw gateway
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl enable openclaw
sudo systemctl start openclaw
sudo systemctl status openclaw
```

### Nginx Reverse Proxy

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

Add SSL with Let's Encrypt:
```bash
sudo certbot --nginx -d your-domain.com
```

---

## Next Steps

Now that your OpenClaw is running:

- **Add more platforms**: Install additional plugins (`openclaw plugins list`)
- **Customize prompts**: Edit `agent.systemPrompt` for your use case
- **Build skills**: Create automation scripts with OpenClaw's skill system
- **Monitor**: Set up alerts for gateway health
- **Backup**: Regularly backup `~/.openclaw/config.json`

---

## Resources

- **Official Docs**: https://docs.openclaw.ai
- **GitHub**: https://github.com/openclaw/openclaw
- **Community Discord**: https://discord.gg/clawd
- **Blog**: https://kunpeng-ai.com/blog/openclaw-getting-started (this guide in Chinese)

---

## Conclusion

OpenClaw puts you back in control. No more monthly subscriptions, no data stored on third-party servers, and one AI assistant that works everywhere you chat.

Install it today — it really does take 5 minutes.

---

*Originally published on [鲲鹏AI探索局](https://kunpeng-ai.com) blog.*
