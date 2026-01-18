[中文](README.md) | [English](README_EN.md) | [日本語](README_JA.md)

---

# Gophish Enhanced Edition - レッドチーム強化版

<div align="center">

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Go Version](https://img.shields.io/badge/go-1.15+-00ADD8.svg)
![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey.svg)

**Gophish をベースにした企業向けフィッシングシミュレーションプラットフォーム**

**レッドチーム評価とセキュリティ意識向上トレーニングに最適化**

[コア機能](#-コア機能) •
[クイックスタート](#-クイックスタート) •
[サーバーデプロイ](#-サーバーデプロイ) •
[機能詳細](#-機能詳細) •
[トラブルシューティング](#-トラブルシューティング)

</div>

---

## 💡 概要

Gophish Enhanced Edition は、オリジナルの Gophish v0.12.1 をベースに深くカスタマイズされたバージョンです。実際のレッドチーム運用や企業セキュリティトレーニングで発生する一般的な問題（メールのブロック、明らかなフィンガープリント、ローカライゼーション不足など）に対応しています。

**なぜ Enhanced Edition を選ぶのか？**

- 🛡️ **高いステルス性**：ウイルス対策ソフトやメールゲートウェイをトリガーするすべてのフィンガープリントヘッダーを削除
- 📧 **高い配信率**：自動 MIME デグレードとネイティブ QR コード技術により、メールが正確に配信・表示されることを保証
- 🌏 **ローカライズ体験**：完全な中国語管理インターフェース
- ⚡ **すぐに使える**：コンパイル後すぐに実行可能、複雑なフロントエンドビルド環境は不要

---

## ✨ コア機能

オリジナル Gophish と比較して、Enhanced Edition は以下の画期的な改善を実装しています：

| 次元 | オリジナルの問題点 | Enhanced ソリューション |
|------|-------------------|------------------------|
| **フィンガープリント** | `X-Gophish` ヘッダーがツールの身元を露出 | ✅ **ゼロフィンガープリント**：すべての識別ヘッダーを削除、送信者クライアントを偽装（Zoho/Outlook） |
| **QR コード** | リモート画像参照で IP が露出、ブロックされる | ✅ **CID 埋め込み**：QR コードを添付ファイルとして送信、**オフラインで表示可能、100% 表示率** |
| **配信** | プレーンテキストがないためスパム判定される | ✅ **スマートデグレード**：HTML を自動的にプレーンテキストに変換、RFC 準拠 |
| **インターフェース** | 英語のみの UI | ✅ **深いローカライゼーション**：コアページ（キャンペーン、ユーザー、テンプレート）を完全ローカライズ |
| **アンチトレース** | デフォルト 404 ページと固定ルートが追跡されやすい | ✅ **動的カモフラージュ**：カスタム 404 ページ、ルートを静的リソースに偽装（.js, .png） |

---

## 🚀 クイックスタート

### オプション 1：コンパイル済みバイナリをダウンロード（推奨）

[Releases](https://github.com/25smoking/GophisModified/releases) からプラットフォームに対応したバイナリをダウンロード：

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

### オプション 2：ソースからビルド

```bash
# 1. プロジェクトをクローン
git clone https://github.com/25smoking/GophisModified.git
cd GophisModified

# 2. ビルド
go build -o gophish

# 3. 実行
./gophish
```

> **注意**：初回実行時に SSL 証明書とデータベースが自動生成されます。初期管理者パスワードはターミナル出力を確認してください。

---

## 🖥️ サーバーデプロイ

### 新規インストール（Linux サーバー）

Gophish をインストールしたことがないサーバー向け。

#### 1. ファイルをサーバーにアップロード

コンパイル済みファイルをパッケージ化してサーバーにアップロード：

```bash
# ローカルでパッケージ化
zip -r gophish-v4.zip gophish config.json static/ templates/ db/

# サーバーにアップロード
scp gophish-v4.zip root@your-server:/tmp/
```

#### 2. サーバー側デプロイ

```bash
# サーバーにログイン、root に切り替え
sudo -i

# ディレクトリを作成して解凍
mkdir -p /opt/gophish
cd /opt/gophish
unzip -o /tmp/gophish-v4.zip -d .

# 実行権限を付与
chmod +x gophish

# ファイアウォールを設定
ufw allow 3333/tcp  # 管理パネル
ufw allow 80/tcp    # フィッシングサーバー

# テスト実行
./gophish
```

#### 3. Systemd サービスをセットアップ（バックグラウンド実行）

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

# サービスを開始
systemctl daemon-reload
systemctl enable gophish
systemctl start gophish

# 初期パスワードを確認
grep "Please login" /var/log/gophish.log
```

---

### オリジナルバージョンからのアップグレード

既に公式 Gophish v0.12.1 を実行している環境で、既存データを保持しながら Enhanced Edition にシームレスにアップグレードする場合。

> ⚠️ **重要**：Enhanced Edition はデータベース構造を変更しています（`qr_size` フィールドを追加）。**データベースマイグレーションが必須**です。

#### 1. データをバックアップ

```bash
systemctl stop gophish
cd /opt/gophish
cp gophish.db gophish.db.bak_$(date +%F)
cp config.json config.json.bak_$(date +%F)
```

#### 2. ファイルを上書き

```bash
# データベースファイルを除外して解凍、既存データを保持
unzip -o /tmp/gophish-v4.zip -x "gophish.db" -d /opt/gophish/
chmod +x /opt/gophish/gophish
```

#### 3. データベースマイグレーション（重要なステップ）

```bash
# sqlite3 をインストール（未インストールの場合）
apt install sqlite3 -y

# 新しいフィールドを追加
sqlite3 gophish.db "ALTER TABLE campaigns ADD COLUMN qr_size TEXT;"

# マイグレーション結果を確認
sqlite3 gophish.db "PRAGMA table_info(campaigns);" | grep qr_size
```

#### 4. サービスを再起動

```bash
systemctl start gophish
systemctl status gophish
```

---

## 🔍 機能詳細

### 1. ネイティブ QR コード（CID 埋め込み）

コア機能の一つ。従来のフィッシングメールは `<img src="http://...">` を使用して QR コードを挿入しますが、ブロックされることが多いです。

**Enhanced メカニズム**：**MIME CID (Content-ID)** メカニズムを使用。QR コードは送信時に即座に生成され、**埋め込み添付ファイル**として送信されます。

**使用方法**：
1. キャンペーン作成時、**「QR コードサイズ」**フィールドを設定（推奨：256）
2. メールテンプレートで `{{.QR}}` 変数を使用

**メリット**：
- 💯 **100% 表示率**：画像はメールの一部なので、オフラインでも表示可能
- 🔒 **より安全**：リモート画像読み込み警告をトリガーしない

### 2. メールフィンガープリント削除（ステルスモード）

- `X-Gophish` で始まるすべてのヘッダーを完全に削除
- `X-Mailer` を一般的なメールクライアントに偽装（Microsoft Outlook 16.0 / Zoho Mail）

### 3. ローカライズされた管理インターフェース

最も頻繁に使用されるモジュールをカバー：
- **キャンペーン**：作成、編集、ダッシュボード
- **ユーザーとグループ**：ターゲットのインポート、編集
- **メールテンプレート**：ビジュアルエディター
- **ランディングページ**：ページエディター

---

## ⚙️ 設定（config.json）

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

## 🔧 トラブルシューティング

### Q: 起動エラー `table campaigns has no column named qr_size`
**A**: オリジナルバージョンからアップグレードしたがデータベースマイグレーションを実行していません。以下を実行：
```bash
sqlite3 gophish.db "ALTER TABLE campaigns ADD COLUMN qr_size TEXT;"
```

### Q: ページが回転し続け、読み込まれない
**A**: 考えられる原因：
1. **フロントエンドファイルが不足**：`static/js/dist/` ディレクトリが存在するか確認
2. **ブラウザキャッシュ**：`Ctrl+Shift+R` で強制リフレッシュ、またはシークレットモードを使用
3. **API エラー**：ブラウザ F12 開発者ツールを開いて Network タブを確認

### Q: ブラウザコンソールエラー `user is not defined`
**A**: `templates/base.html` ファイルが破損しているか、バージョンが間違っています。再度上書きしてください。

### Q: テンプレート保存時に `can't evaluate field QR` エラー
**A**: プログラムが再コンパイルされていません。`go build` を実行してサービスを再起動してください。

---

## 🛡️ セキュリティポリシー（必読）

⚠️ **免責事項**：
このプロジェクトは、企業のセキュリティ構築、レッドチーム運用、セキュリティ意識向上トレーニングの**許可された**使用のみを目的としています。

1. **許可が必要**：テストにこのツールを使用する前に、ターゲットシステム所有者から書面による許可を得る必要があります
2. **違法使用禁止**：このツールを許可されていない攻撃活動に使用することは固く禁じられています
3. **データプライバシー**：演習データは匿名化された情報を含む必要があります。演習後はデータベースファイルを速やかに破棄してください

開発者は、このツールの使用に起因する直接的または間接的な結果について一切の責任を負いません。

---

## 🤝 貢献とサポート

バグ報告や提案の Issue を歓迎いたします。

- **GitHub**: [github.com/25smoking/GophisModified](https://github.com/25smoking/GophisModified)

---

<div align="center">

Made with ❤️ for Security Professionals

</div>
