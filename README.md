# MCP Mail Server (Fixed Fork)

![NPM Version](https://img.shields.io/npm/v/mcp-mail-server)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Language:** English | [中文](README-zh.md)

**Note: This is a fork maintained to fix hardcoded timezones and optimize for Raspberry Pi/LibreChat environments.**

A Model Context Protocol server for IMAP/SMTP email operations with Claude, Cursor, and other AI assistants.

## Features

- **IMAP Operations**: Search, read, and manage emails across mailboxes
- **SMTP Support**: Send emails with HTML/text content and attachments  
- **Secure Configuration**: Environment-based setup with TLS/SSL support
- **AI-Friendly**: Natural language commands for email operations
- **Auto Connection Management**: Automatic IMAP/SMTP connection handling
- **Multi-Mailbox Support**: Access INBOX, Sent, and custom folders
- **Dynamic Timezone**: Supports `TIMEZONE` and `LOCALE` environment variables (Fixed in this fork).

---

## Quickstart: Dockerized LibreChat on Raspberry Pi 5 (ARM64)

This guide details how to install the `mcp-mail-server` for a LibreChat instance running on a Raspberry Pi 5. It uses a **Satellite Relay** configuration for maximum reliability and better handling of outbound mail.

### 1. Prerequisite: Host-Side Satellite Relay (Postfix)
Instead of the Docker container connecting directly to external mail servers, we configure the Raspberry Pi host to act as a secure relay.

#### A. Install Postfix on the Pi
Run these commands on your Raspberry Pi terminal:
```bash
sudo apt-get update
sudo apt-get install postfix libsasl2-modules
```
*When prompted, choose **"Satellite system"**.*

#### B. Configure Postfix
Edit `/etc/postfix/main.cf`:
```conf
# Relay through iCloud (example)
relayhost = [smtp.mail.me.com]:587

# SASL Authentication
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_use_tls = yes

# Networking
inet_interfaces = all
# Allow the local host AND the Docker bridge networks
# (Verify your subnet with 'docker network inspect bridge')
mynetworks = 127.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
```

#### C. Set Relay Credentials
```bash
echo "[smtp.mail.me.com]:587 your-email@icloud.com:your-app-specific-password" | sudo tee /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd
sudo chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
sudo systemctl restart postfix
```

### 2. Update `Dockerfile.api`
Since the Raspberry Pi uses ARM64 and LibreChat uses Alpine, build the MCP server natively during the image creation to ensure architecture compatibility:

```dockerfile
FROM ghcr.io/danny-avila/librechat:latest

USER root
# Install build tools and Git
RUN apk add --no-cache git npm make g++ python3

# Clone and build the Mail MCP server
RUN mkdir -p /app/mcp/email-server
RUN git clone https://github.com/ian1roberts/mcp-mail-server.git /app/mcp/email-server && \
    cd /app/mcp/email-server && \
    npm install && \
    npm run build

USER node
WORKDIR /app
CMD ["npm", "run", "backend"]
```

### 3. Set Environment Variables
To keep your credentials secure, do not hardcode them in the YAML. Add your iCloud **App-Specific Password** to your LibreChat `.env` file:

```env
# .env file
ICLOUD_APP_PASS=xxxx-xxxx-xxxx-xxxx
```

### 4. Configure `librechat.yaml`
Point the `email-tool` to the host's relay (the Docker Gateway IP) and reference your environment variable.

```yaml
mcpServers:
  email-tool:
    command: "node"
    args: ["/app/mcp/email-server/dist/index.js"]
    env:
      # Connection to Host Satellite Relay (unencrypted local hop)
      SMTP_HOST: "172.18.0.1" 
      SMTP_PORT: "25"
      SMTP_SECURE: "false" 
      
      # Direct IMAP for receiving (iCloud)
      IMAP_HOST: "imap.mail.me.com"
      IMAP_PORT: "993"
      IMAP_SECURE: "true"
      
      # Identity
      EMAIL_USER: "your-email@icloud.com"
      EMAIL_PASS: "${ICLOUD_APP_PASS}"
      TIMEZONE: "Europe/London" 
      LOCALE: "en-GB"
```

---

## Example Usage

### 1. Checking & Summarizing Mail
*   **The Quick Catch-up:** *"Check my inbox for any new messages from the last 4 hours and give me a bulleted summary of each."*
*   **The Deep Search:** *"Find all emails from 'Amazon' received this week that mention 'Refund' and tell me the total amount."*

### 2. Sending & Drafting
*   **The Professional Follow-up:** *"Send an email to recipient@example.com. Subject: Meeting Follow-up. Body: Hi, it was great catching up. I've started the agentic workflow we discussed. Have a great evening!"*
*   **The Smart Reply:** *"Find the unread email from 'Travel Agency,' read the flight options they sent, and draft a reply choosing the cheapest morning flight."*

### 3. Inbox Management & Automation
*   **The Clean-up Crew:** *"Find all unread newsletters or promotional emails older than 30 days and delete them."*
*   **The Task Master:** *"Search for any emails I received yesterday that I haven't replied to yet. List them by sender and subject."*

---

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

## Remarks, Assumptions, and Security Considerations

- **Network Scope**: The `mynetworks` setting in Postfix is broad. Ensure your Docker network is secure and not accessible from the public internet.
- **Port 25**: We assume Port 25 for the "local hop" between the container and the Pi host. Verify connectivity with `nc -zv 172.18.0.1 25`.
- **Docker Gateway**: Verify your gateway IP using `ip addr show docker0` on the host. If your Docker network differs, update the `SMTP_HOST` accordingly.
- **ARM64 vs. x86_64**: This guide is specifically optimized for the **Raspberry Pi 5 (ARM64)** using Alpine Linux.

## Acknowledgements

We would like to express our gratitude to the original authors and contributors of the [mcp-mail-server](https://github.com/yunfeizhu/mcp-mail-server) repository. This fork builds upon their excellent foundation to provide enhanced timezone support and specialized deployment guides for the LibreChat and Raspberry Pi communities.

## License

MIT License - see [LICENSE](LICENSE) file for details.
