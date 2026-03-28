# Inception

42 Tokyo のシステム管理課題「Inception」の実装です。Docker を使用して Nginx・WordPress・MariaDB を独立したコンテナで構築し、LEMP スタックを仮想環境上に展開することを目的としています。

---

## 目的

- Docker および Docker Compose を用いたマルチコンテナ構成の理解と実装
- Nginx による HTTPS (TLS 1.2/1.3) リバースプロキシの構築
- PHP-FPM を使った WordPress の動作と MariaDB との連携
- 各サービスを独立した Dockerfile から独自ビルドし、Docker Hub のイメージに依存しない設計

---

## 概要

| サービス | 役割 |
|----------|------|
| **Nginx** | HTTPS (443) で受け付け、PHP リクエストを WordPress (PHP-FPM) へ転送するリバースプロキシ |
| **WordPress** | PHP 7.4-FPM で動作する CMS。MariaDB をバックエンド DB として利用 |
| **MariaDB** | WordPress 用のデータベースサーバー。初回起動時にスクリプトで DB・ユーザーを自動作成 |

各コンテナは独立した Dockerfile からビルドされ、Docker Compose で一括管理されます。コンテナ間の通信はブリッジネットワーク (`network`) を介し、永続データは bind mount ボリュームでホストに保存されます。

---

## ディレクトリ構成

```
inception/
├── Makefile                          # ビルド・起動・停止・クリーンアップ用コマンド集
└── scrs/
    ├── .env                          # 環境変数 (DB名・ユーザー・パスワード等)
    ├── docker-compose.yml            # サービス定義・ネットワーク・ボリューム設定
    └── requirements/
        ├── nginx/
        │   ├── Dockerfile            # Nginx + OpenSSL イメージのビルド定義
        │   └── conf/
        │       └── nginx.conf        # SSL設定・fastcgi_pass による PHP-FPM 転送設定
        ├── wordpress/
        │   ├── Dockerfile            # PHP 7.4-FPM + WordPress 実行環境のビルド定義
        │   ├── conf/
        │   │   └── www.conf          # PHP-FPM プール設定 (listen: 0.0.0.0:9000)
        │   └── tools/
        │       └── create_wordpress.sh  # WordPress のダウンロードと wp-config.php 生成スクリプト
        └── mariadb/
            ├── Dockerfile            # MariaDB サーバーイメージのビルド定義
            ├── conf/
            │   ├── mysqld.conf       # mysqld 設定ファイル (参考用)
            │   └── wordpress.sql     # WordPress DB のダンプファイル
            └── tools/
                └── mariadb.sh        # DB初期化・ユーザー作成スクリプト
```

---

## 技術スタック

| 分類 | 技術 |
|------|------|
| コンテナ管理 | Docker、Docker Compose |
| ベース OS | Debian Buster (Nginx)、Debian Bullseye (WordPress・MariaDB) |
| Web サーバー / TLS | Nginx、OpenSSL (自己署名証明書)、TLS 1.2/1.3 |
| アプリケーション | WordPress、PHP 7.4-FPM |
| データベース | MariaDB |
| スクリプト | POSIX sh |

---

## セットアップ・起動方法

### 事前準備

1. ホスト側のデータディレクトリを作成する

```bash
mkdir -p /home/kajid777/data/mysql
mkdir -p /home/kajid777/data/wordpress
```

2. `/etc/hosts` にドメインを追加する

```
127.0.0.1 kajid777.42.fr
```

3. `scrs/.env` に必要な環境変数を設定する

```
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_DATABASE=wordpress
MYSQL_USER=your_user
MYSQL_PASSWORD=your_password
```

### コマンド

```bash
# コンテナのビルドと起動
make

# コンテナの停止
make down

# 強制再ビルドと起動
make re

# コンテナ・イメージ・ボリューム・ネットワークの全削除
make clean
```

起動後、ブラウザで `https://kajid777.42.fr` にアクセスすると WordPress サイトが表示されます。

---

## ネットワーク・ボリューム

| 種類 | 名前 | 詳細 |
|------|------|------|
| ネットワーク | `network` | ブリッジドライバー。全サービスが同一ネットワーク内で通信 |
| ボリューム | `mariadb_data` | `/home/kajid777/data/mysql` へのバインドマウント |
| ボリューム | `wordpress_data` | `/home/kajid777/data/wordpress` へのバインドマウント |
