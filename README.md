[中文](README.md) | [English](README_EN.md) | [日本語](README_JA.md)

---

# Gophish Enhanced Edition - 红队增强版

<div align="center">

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Go Version](https://img.shields.io/badge/go-1.15+-00ADD8.svg)
![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey.svg)

**基于官方 Gophish 深度二次开发的企业级钓鱼演练平台**

**针对红队评估（Red Teaming）与安全意识培训场景进行全方位优化**

[核心功能](#-核心功能) •
[快速开始](#-快速开始) •
[服务器部署](#-服务器部署) •
[功能详解](#-功能详解) •
[故障排查](#-故障排查)

</div>

---

## 💡 项目简介

Gophish Enhanced Edition 是在原版 Gophish v0.12.1 基础上的深度改进版本。针对原版在实际红队演练和企业安全培训中遇到的痛点（如邮件被拦截、特征明显、缺乏本地化等），进行了大量底层代码级的优化和增强。

**为什么选择增强版？**

- 🛡️ **高隐蔽性**：移除了所有导致邮件被杀毒软件或网关拦截的指纹特征
- 📧 **高送达率**：通过自动化 MIME 降级和原生二维码技术，确保邮件能准确送达并展示
- 🌏 **本土化体验**：全中文管理界面，降低操作门槛，适合国内安全团队使用
- ⚡ **开箱即用**：编译后即可运行，无需复杂的前端构建环境

---

## ✨ 核心功能

相比原版 Gophish，增强版实现了以下突破性改进：

| 增强维度 | 原版痛点 | 增强版方案 |
|---------|---------|-----------|
| **特征隐藏** | `X-Gophish` 头部直接暴露工具身份 | ✅ **零特征**：移除所有标识头，伪装发件客户端（Zoho/Outlook） |
| **二维码投放** | 只能引用远程图片，不仅暴露 IP 且容易被拦截不显示 | ✅ **CID 内嵌技术**：二维码作为附件随邮件下发，**无需联网加载，必显** |
| **邮件投递** | 常常因缺少纯文本内容部分被判定为垃圾邮件 | ✅ **智能降级**：自动将 HTML 内容转化为纯文本，符合 RFC 标准，提高信誉度 |
| **用户界面** | 全英文界面，对非技术人员不友好 | ✅ **深度汉化**：活动、模板、用户管理等核心页面全中文支持 |
| **反溯源** | 默认 404 页面和固定路由容易被蓝队溯源 | ✅ **动态伪装**：支持自定义 404 页面，路由伪装成静态资源（如 `.js`, `.png`） |

---

## 🚀 快速开始

### 方式一：下载预编译版本（推荐）

从 [Releases](https://github.com/25smoking/GophisModified/releases) 页面下载对应平台的二进制文件：

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

### 方式二：从源码编译

```bash
# 1. 克隆项目
git clone https://github.com/25smoking/GophisModified.git
cd GophisModified

# 2. 编译
go build -o gophish

# 3. 运行
./gophish
```

> **注意**：首次运行时会自动生成 SSL 证书和数据库，请查看终端输出获取初始管理员密码。

---

## 🖥️ 服务器部署

### 全新安装（Linux 服务器）

适用于从未安装过 Gophish 的新服务器。

#### 1. 上传文件到服务器

将编译好的文件打包上传至服务器：

```bash
# 在本地打包
zip -r gophish-v4.zip gophish config.json static/ templates/ db/

# 上传到服务器
scp gophish-v4.zip root@your-server:/tmp/
```

#### 2. 服务器端部署

```bash
# 登录服务器，切换到 root 用户
sudo -i

# 创建目录并解压
mkdir -p /opt/gophish
cd /opt/gophish
unzip -o /tmp/gophish-v4.zip -d .

# 赋予执行权限
chmod +x gophish

# 配置防火墙
ufw allow 3333/tcp  # 管理后台
ufw allow 80/tcp    # 钓鱼服务

# 测试运行
./gophish
```

#### 3. 设置 Systemd 服务（后台运行）

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

# 启动服务
systemctl daemon-reload
systemctl enable gophish
systemctl start gophish

# 查看初始密码
grep "Please login" /var/log/gophish.log
```

---

### 从原版升级

适用于已经运行官方 Gophish v0.12.1 的环境，希望无缝升级到增强版，保留原有数据。

> ⚠️ **重要提示**：增强版修改了数据库结构（增加了 `qr_size` 字段）。**必须执行数据库迁移**，否则会报错。

#### 1. 备份数据

```bash
systemctl stop gophish
cd /opt/gophish
cp gophish.db gophish.db.bak_$(date +%F)
cp config.json config.json.bak_$(date +%F)
```

#### 2. 覆盖文件

```bash
# 解压时排除数据库文件，保留原有数据
unzip -o /tmp/gophish-v4.zip -x "gophish.db" -d /opt/gophish/
chmod +x /opt/gophish/gophish
```

#### 3. 数据库迁移（关键步骤）

```bash
# 安装 sqlite3（如果没有）
apt install sqlite3 -y

# 添加新字段
sqlite3 gophish.db "ALTER TABLE campaigns ADD COLUMN qr_size TEXT;"

# 验证迁移结果
sqlite3 gophish.db "PRAGMA table_info(campaigns);" | grep qr_size
```

#### 4. 重启服务

```bash
systemctl start gophish
systemctl status gophish
```

---

## 🔍 功能详解

### 1. 原生二维码 (CID Embedded)

这是本版本最核心的特性之一。传统的钓鱼邮件插入二维码通常使用 `<img src="http://...">`，容易被拦截不显示。

**增强版机制**：使用 **MIME CID (Content-ID)** 机制，二维码在发送时即时生成，作为邮件的**内嵌附件**发送。

**使用方法**：
1. 创建活动时，设置 **"二维码大小"** 字段（推荐 256）
2. 在邮件模板中使用 `{{.QR}}` 变量

**优势**：
- 💯 **100% 展示率**：图片是邮件的一部分，离线也能显示
- 🔒 **更安全**：不触发远程图片加载警告

### 2. 邮件指纹消除 (Stealth Mode)

- 彻底移除所有 `X-Gophish` 开头的头部字段
- 将 `X-Mailer` 伪装成常见邮件客户端（Microsoft Outlook 16.0 / Zoho Mail）

### 3. 中文管理界面

覆盖日常操作最频繁的模块：
- **活动管理**：新建、编辑、结果仪表盘
- **用户与组**：目标导入、编辑
- **邮件模板**：可视化编辑器
- **钓鱼页面**：页面编辑器

---

## ⚙️ 核心配置 (config.json)

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

## 🔧 故障排查

### Q: 启动报错 `table campaigns has no column named qr_size`
**A**: 您从原版升级但没有执行数据库迁移。请运行：
```bash
sqlite3 gophish.db "ALTER TABLE campaigns ADD COLUMN qr_size TEXT;"
```

### Q: 页面一直转圈，无法加载
**A**: 可能原因：
1. **前端文件缺失**：检查 `static/js/dist/` 目录是否存在
2. **浏览器缓存**：按 `Ctrl+Shift+R` 强制刷新，或使用无痕模式
3. **API 错误**：打开浏览器 F12 开发者工具查看 Network 标签

### Q: 浏览器控制台报错 `user is not defined`
**A**: `templates/base.html` 文件损坏或版本不对，请重新覆盖。

### Q: 保存模板时报错 `can't evaluate field QR`
**A**: 程序未重新编译。请执行 `go build` 后重启服务。

---

## 🛡️ 安全规范 (必读)

⚠️ **免责声明**：
本项目仅供授权的企业安全建设、红队演练和安全意识培训使用。

1. **必须获得授权**：在使用本工具进行任何测试前，必须获得目标系统所有者的书面授权
2. **严禁非法使用**：禁止将本工具用于任何未授权的攻击行为
3. **数据隐私**：演练数据应包含脱敏信息，演练结束后请及时销毁数据库文件

开发者不对因使用本工具造成的任何直接或间接后果承担责任。

---

## 🤝 贡献与支持

欢迎提交 Issue 反馈 Bug 或建议。

- **GitHub**: [github.com/25smoking/GophisModified](https://github.com/25smoking/GophisModified)

---

<div align="center">

Made with ❤️ for Security Professionals

</div>
