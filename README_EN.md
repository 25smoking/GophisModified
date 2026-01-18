[中文](README.md) | [English](README_EN.md) | [日本語](README_JA.md)

---

# Gophish Enhanced Edition

<div align="center">

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Go Version](https://img.shields.io/badge/go-1.15+-00ADD8.svg)
![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey.svg)

**Enterprise-Grade Phishing Simulation Platform based on Gophish**

**Optimized for Red Teaming & Security Awareness Training**

[Core Features](#-core-features) •
[Quick Start](#-quick-start) •
[Server Deployment](#-server-deployment) •
[Feature Details](#-feature-details) •
[Troubleshooting](#-troubleshooting)

</div>

---

## 💡 Introduction

Gophish Enhanced Edition is a deeply customized version based on the original Gophish v0.12.1. It addresses common pain points encountered in real-world red team operations and corporate security training, such as email blocking, obvious fingerprints, and lack of localization.

**Why Enhanced Edition?**

- 🛡️ **High Stealth**: Removes all fingerprint headers that trigger antivirus software or email gateways
- 📧 **High Deliverability**: Ensures emails are delivered and displayed correctly using automated MIME degradation and native QR code technology
- 🌏 **Localized Experience**: Full Chinese management interface, lowering the barrier for entry
- ⚡ **Ready to Use**: Runs immediately after compilation, no complex frontend build environment required

---

## ✨ Core Features

Compared to the original Gophish, the Enhanced Edition implements the following breakthrough improvements:

| Dimension | Original Pain Point | Enhanced Solution |
|-----------|---------------------|-------------------|
| **Fingerprints** | `X-Gophish` headers expose tool identity | ✅ **Zero Fingerprint**: Removes all ID headers, spoofs sender clients (Zoho/Outlook) |
| **QR Code** | Remote image reference exposes IP & gets blocked | ✅ **CID Embedded**: QR codes sent as attachments, **offline viewable, 100% display rate** |
| **Delivery** | Often flagged as spam due to missing plain text | ✅ **Smart Degradation**: Auto-converts HTML to plain text, RFC standard compliant |
| **Interface** | English-only UI, unfriendly to some users | ✅ **Deep Localization**: Core pages (Campaigns, Users, Templates) fully localized |
| **Anti-Tracing** | Default 404 page & fixed routes are easily traced | ✅ **Dynamic Camouflage**: Custom 404 pages, routes disguised as static resources (.js, .png) |

---

## 🚀 Quick Start

### Option 1: Download Pre-compiled Binary (Recommended)

Download the binary for your platform from [Releases](https://github.com/25smoking/GophisModified/releases):

```bash
# Linux
wget https://github.com/25smoking/GophisModified/releases/latest/download/gophish-linux-amd64.zip
unzip gophish-linux-amd64.zip
chmod +x gophish
./gophish

# macOS
wget https://github.com/25smoking/GophisModified/releases/latest/download/gophish-darwin-amd64.zip
unzip gophish-darwin-amd64.zip
chmod +x gophish
./gophish
```

### Option 2: Build from Source

```bash
# 1. Clone project
git clone https://github.com/25smoking/GophisModified.git
cd GophisModified

# 2. Build
go build -o gophish

# 3. Run
./gophish
```

> **Note**: On first run, SSL certificates and database will be auto-generated. Check terminal output for the initial admin password.

---

## 🖥️ Server Deployment

### Fresh Installation (Linux Server)

For servers that have never had Gophish installed.

#### 1. Upload Files to Server

Package and upload the compiled files to your server:

```bash
# Package locally
zip -r gophish-v4.zip gophish config.json static/ templates/ db/

# Upload to server
scp gophish-v4.zip root@your-server:/tmp/
```

#### 2. Server-side Deployment

```bash
# Login to server, switch to root
sudo -i

# Create directory and extract
mkdir -p /opt/gophish
cd /opt/gophish
unzip -o /tmp/gophish-v4.zip -d .

# Grant execute permission
chmod +x gophish

# Configure firewall
ufw allow 3333/tcp  # Admin panel
ufw allow 80/tcp    # Phishing server

# Test run
./gophish
```

#### 3. Setup Systemd Service (Background Running)

```bash
cat > /etc/systemd/system/gophish.service << EOF
[Unit]
Description=Gophish Enhanced Edition
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/gophish
ExecStart=/opt/gophish/gophish
Restart=always
RestartSec=5
StandardOutput=append:/var/log/gophish.log
StandardError=append:/var/log/gophish.log

[Install]
WantedBy=multi-user.target
EOF

# Start service
systemctl daemon-reload
systemctl enable gophish
systemctl start gophish

# Check initial password
grep "Please login" /var/log/gophish.log
```

---

### Upgrade from Original Version

For environments already running official Gophish v0.12.1, wanting to seamlessly upgrade to Enhanced Edition while preserving existing data.

> ⚠️ **Important**: Enhanced Edition modifies the database structure (adds `qr_size` field). **Database migration is required**, otherwise errors will occur.

#### 1. Backup Data

```bash
systemctl stop gophish
cd /opt/gophish
cp gophish.db gophish.db.bak_$(date +%F)
cp config.json config.json.bak_$(date +%F)
```

#### 2. Overwrite Files

```bash
# Extract excluding database file to preserve existing data
unzip -o /tmp/gophish-v4.zip -x "gophish.db" -d /opt/gophish/
chmod +x /opt/gophish/gophish
```

#### 3. Database Migration (Critical Step)

```bash
# Install sqlite3 (if not present)
apt install sqlite3 -y

# Add new field
sqlite3 gophish.db "ALTER TABLE campaigns ADD COLUMN qr_size TEXT;"

# Verify migration result
sqlite3 gophish.db "PRAGMA table_info(campaigns);" | grep qr_size
```

#### 4. Restart Service

```bash
systemctl start gophish
systemctl status gophish
```

---

## 🔍 Feature Details

### 1. Native QR Code (CID Embedded)

One of the core features. Traditional phishing emails insert QR codes using `<img src="http://...">`, which often gets blocked.

**Enhanced Mechanism**: Uses **MIME CID (Content-ID)** mechanism. The QR code is generated instantly upon sending and sent as an **embedded attachment**.

**Usage**:
1. When creating a campaign, set the **"QR Code Size"** field (recommended: 256)
2. Use the `{{.QR}}` variable in your email template

**Advantages**:
- 💯 **100% Display Rate**: Image is part of the email, viewable offline
- 🔒 **More Secure**: Doesn't trigger remote image loading warnings

### 2. Email Fingerprint Removal (Stealth Mode)

- Completely removes all headers starting with `X-Gophish`
- Spoofs `X-Mailer` to appear as common email clients (Microsoft Outlook 16.0 / Zoho Mail)

### 3. Localized Management Interface

Covers the most frequently used modules:
- **Campaigns**: Creation, Editing, Dashboard
- **Users & Groups**: Import targets, Editing
- **Email Templates**: Visual Editor
- **Landing Pages**: Page Editor

---

## ⚙️ Configuration (config.json)

```json
{
  "admin_server": {
    "listen_url": "127.0.0.1:3333",
    "use_tls": true,
    "cert_path": "gophish_admin.crt",
    "key_path": "gophish_admin.key"
  },
  "phish_server": {
    "listen_url": "0.0.0.0:80",
    "use_tls": false
  },
  "db_name": "sqlite3",
  "db_path": "gophish.db",
  "migrations_prefix": "db/db_"
}
```

---

## 🔧 Troubleshooting

### Q: Startup error `table campaigns has no column named qr_size`
**A**: You upgraded from the original version without database migration. Run:
```bash
sqlite3 gophish.db "ALTER TABLE campaigns ADD COLUMN qr_size TEXT;"
```

### Q: Page keeps spinning, won't load
**A**: Possible causes:
1. **Missing frontend files**: Check if `static/js/dist/` directory exists
2. **Browser cache**: Press `Ctrl+Shift+R` for hard refresh, or use incognito mode
3. **API error**: Open browser F12 developer tools and check Network tab

### Q: Browser console error `user is not defined`
**A**: `templates/base.html` file is corrupted or wrong version, please re-overwrite.

### Q: Saving template shows `can't evaluate field QR`
**A**: Program not recompiled. Run `go build` and restart the service.

---

## 🛡️ Security Policy (Must Read)

⚠️ **Disclaimer**:
This project is intended ONLY for authorized enterprise security construction, red team operations, and security awareness training.

1. **Authorization Required**: You must obtain written authorization from the target system owner before using this tool for any testing
2. **Illegal Use Prohibited**: Using this tool for any unauthorized attack activities is strictly prohibited
3. **Data Privacy**: Exercise data should contain desensitized information; please destroy database files promptly after the exercise

The developers assume no liability for any direct or indirect consequences resulting from the use of this tool.

---

## 🤝 Contribution & Support

Issues for bug reports or suggestions are welcome.

- **GitHub**: [github.com/25smoking/GophisModified](https://github.com/25smoking/GophisModified)

---

<div align="center">

Made with ❤️ for Security Professionals

</div>
