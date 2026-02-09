# MCP Mail Server (Fixed Fork)

![NPM Version](https://img.shields.io/npm/v/mcp-mail-server)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Language:** English | [中文](README-zh.md)

**Note: This is a fork maintained to fix hardcoded timezones.**

A Model Context Protocol server for IMAP/SMTP email operations with Claude, Cursor, and other AI assistants.

## Features

- **IMAP Operations**: Search, read, and manage emails across mailboxes
- **SMTP Support**: Send emails with HTML/text content and attachments  
- **Secure Configuration**: Environment-based setup with TLS/SSL support
- **AI-Friendly**: Natural language commands for email operations
- **Auto Connection Management**: Automatic IMAP/SMTP connection handling
- **Multi-Mailbox Support**: Access INBOX, Sent, and custom folders
- **Dynamic Timezone**: Supports `TIMEZONE` and `LOCALE` environment variables (Fixed in this fork).

## Quick Start

1. **Install**: `npx -y github:ian1roberts/mcp-mail-server`
2. **Configure** environment variables (see [Configuration](#configuration))
3. **Add** to your MCP client configuration
4. **Use** natural language: *"Show me unread emails from today"*

## Installation

<details>
<summary>LibreChat (Docker/YAML)</summary>

Add to your `librechat.yaml`:

```yaml
mcpServers:
  email-tool:
    command: "npx"
    args: ["-y", "github:ian1roberts/mcp-mail-server"]
    env:
      IMAP_HOST: "your-imap-server.com"
      IMAP_PORT: "993"
      IMAP_SECURE: "true"
      SMTP_HOST: "your-smtp-server.com"
      SMTP_PORT: "465"
      SMTP_SECURE: "true"
      EMAIL_USER: "your-email@domain.com"
      EMAIL_PASS: "your-password"
      TIMEZONE: "Europe/London" # Custom Timezone support
      LOCALE: "en-GB"
```

</details>

<details>
<summary>Claude Desktop</summary>

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcp-mail-server": {
      "command": "npx",
      "args": ["-y", "github:ian1roberts/mcp-mail-server"],
      "env": {
        "IMAP_HOST": "your-imap-server.com",
        "IMAP_PORT": "993",
        "IMAP_SECURE": "true",
        "SMTP_HOST": "your-smtp-server.com",
        "SMTP_PORT": "465",
        "SMTP_SECURE": "true",
        "EMAIL_USER": "your-email@domain.com",
        "EMAIL_PASS": "your-password",
        "TIMEZONE": "Europe/London"
      }
    }
  }
}
```

</details>

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `IMAP_HOST` | IMAP server address | Required |
| `IMAP_PORT` | IMAP port number | Required |
| `IMAP_SECURE` | Enable TLS | Required |
| `SMTP_HOST` | SMTP server address | Required |
| `SMTP_PORT` | SMTP port number | Required |
| `SMTP_SECURE` | Enable SSL | Required |
| `EMAIL_USER` | Email username | Required |
| `EMAIL_PASS` | Email password/app password | Required |
| `TIMEZONE` | System timezone | System default (fixed from Asia/Shanghai) |
| `LOCALE` | System locale | en-US (fixed from zh-CN) |

## Development

1. **Clone the repository**:
   ```bash
   git clone https://github.com/ian1roberts/mcp-mail-server.git
   cd mcp-mail-server
   ```

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

**Fork Information:**
- Repository: [ian1roberts/mcp-mail-server](https://github.com/ian1roberts/mcp-mail-server)
- Original Author: zhuyunfei
