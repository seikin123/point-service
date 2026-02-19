# Point Service

ポイント・キャッシュバックサービスのプロジェクト

## 技術スタック

### バックエンド
- Laravel 11
- MySQL 8.4
- PHP 8.3

### フロントエンド
- Vue 3 (Composition API)
- TypeScript
- Tailwind CSS v4
- Inertia.js
- Vite

### 開発環境
- Docker (Laravel Sail)
- ESLint + Prettier
- Vitest

## 環境構築手順

### 前提条件

以下がインストールされていることを確認してください。

- **Docker Desktop**
  - Mac: https://docs.docker.com/desktop/install/mac-install/
  - Windows: https://docs.docker.com/desktop/install/windows-install/
- **Git**
  - インストール確認: `git --version`
  - Mac: Xcodeコマンドラインツールに含まれる
  - Windows: https://git-scm.com/download/win

---

### ステップ1: リポジトリのクローン

```bash
# リポジトリをクローン
git clone https://github.com/seikin123/point-service.git

# プロジェクトディレクトリに移動
cd point-service
```

---

### ステップ2: 環境変数ファイルの作成

```bash
# .env.exampleから.envを作成
cp .env.example .env
```

`.env` ファイルを開いて以下を確認・編集してください。

```env
APP_NAME=PointService
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_TIMEZONE=Asia/Tokyo
APP_URL=http://localhost

# データベース設定
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=sail
DB_PASSWORD=password

# ローカルのMySQLと競合する場合は以下を追加
FORWARD_DB_PORT=3307

# キャッシュ・セッション設定
CACHE_STORE=file
SESSION_DRIVER=database
QUEUE_CONNECTION=sync

# メール設定（開発環境）
MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
```

---

### ステップ3: Composerパッケージのインストール

**初回のみ必要な手順です。** Sailを使う前にComposerの依存関係をインストールします。

```bash
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php83-composer:latest \
    composer install --ignore-platform-reqs
```

**Windowsの場合:**

```bash
docker run --rm ^
    -v "%cd%":/var/www/html ^
    -w /var/www/html ^
    laravelsail/php83-composer:latest ^
    composer install --ignore-platform-reqs
```

---

### ステップ4: Sailエイリアスの設定（推奨）

毎回 `./vendor/bin/sail` と入力するのは面倒なので、エイリアスを設定します。

**Mac/Linux の場合:**

```bash
# ~/.zshrc または ~/.bashrc に追加
echo "alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'" >> ~/.zshrc

# 設定を反映
source ~/.zshrc
```

**Windowsの場合:**

PowerShellで以下を実行:

```powershell
# プロファイルファイルを開く
notepad $PROFILE

# 以下を追加して保存
function sail { bash vendor/bin/sail $args }
```

---

### ステップ5: Dockerコンテナの起動

```bash
# バックグラウンドでコンテナを起動
sail up -d

# 起動確認（mysql, mailpitが "Up" になっていればOK）
sail ps
```

**初回起動時の注意:**
- Dockerイメージのダウンロードとビルドで5〜10分かかります
- MySQLの初期化に30秒ほどかかります

---

### ステップ6: アプリケーションの初期設定

```bash
# アプリケーションキーの生成
sail artisan key:generate

# データベースマイグレーション実行
sail artisan migrate

# 確認: テーブルが作成されているか
sail mysql -e "SHOW TABLES;"
```

---

### ステップ7: フロントエンドのセットアップ

```bash
# NPMパッケージのインストール
sail npm install

# 脆弱性の自動修正（任意）
sail npm audit fix

# 開発用ビルド（動作確認用）
sail npm run build
```

---

### ステップ8: 開発サーバーの起動

**ターミナルを2つ開いて作業することを推奨します。**

**ターミナル1（Vite開発サーバー - 常時起動）:**

```bash
sail npm run dev
```

以下のような表示が出たら成功です:

```
  VITE v5.x.x  ready in xxx ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  Laravel: http://localhost
```

**ターミナル2（作業用）:**

```bash
# Artisanコマンドなどを実行
sail artisan tinker
```

---

### ステップ9: ブラウザで確認

http://localhost にアクセスしてカウンターが表示されれば成功です！

---

### ステップ10: データベースGUIツールで接続（任意）

TablePlusやDBeaver、MySQL Workbenchなどで接続する場合:

| 項目 | 値 |
|---|---|
| Host | `127.0.0.1` |
| Port | `3307`（または `.env` の `FORWARD_DB_PORT` の値） |
| Database | `laravel` |
| Username | `sail` |
| Password | `password` |

---

## トラブルシューティング

### エラー: `Port 3306 is already in use`

ローカルのMySQLと競合しています。

**解決策:**

1. `.env` に以下を追加:
   ```env
   FORWARD_DB_PORT=3307
   ```

2. コンテナを再起動:
   ```bash
   sail down
   sail up -d
   ```

---

### エラー: `SQLSTATE[HY000] [2002] Connection refused`

MySQLコンテナの起動が完了していません。

**解決策:**

```bash
# MySQLが起動するまで待つ（30秒ほど）
sail ps

# mysqlが "Up (healthy)" になったら再実行
sail artisan migrate
```

---

### エラー: `Access denied for user 'sail'@'localhost'`

MySQLのボリュームに古いデータが残っています。

**解決策:**

```bash
# ボリュームごと削除して再作成
sail down --volumes
sail up -d

# 30秒待ってからマイグレーション
sail artisan migrate
```

---

### エラー: `Vite manifest not found`

Viteのビルドが実行されていません。

**解決策:**

```bash
# ビルドを実行
sail npm run build

# または開発サーバーを起動
sail npm run dev
```

---

### その他のエラー

```bash
# キャッシュをクリア
sail artisan cache:clear
sail artisan config:clear
sail artisan route:clear
sail artisan view:clear

# Composerパッケージの再インストール
sail composer install

# NPMパッケージの再インストール
sail exec laravel.test rm -rf node_modules package-lock.json
sail npm install
```

## 日常の開発フロー

```bash
# 朝、開発開始時
sail up -d
sail npm run dev  # 別ターミナルで起動

# コード編集...

# Lintチェック
sail npm run lint

# テスト実行
sail npm run test        # Vueコンポーネント
sail artisan test        # Laravel

# 夕方、開発終了時
Ctrl + C  # Vite停止
sail down
```

## よく使うコマンド

```bash
# Artisanコマンド
sail artisan migrate
sail artisan tinker

# Composer
sail composer require package-name

# NPM
sail npm install package-name

# データベース接続
sail mysql

# コンテナ内シェル
sail shell

# ログ確認
sail logs
```

## チーム開発のルール

- コミット前に `sail npm run lint:fix` でコード整形
- プルリクエスト作成前に `sail artisan test` と `sail npm run test` を実行
- `.env` はコミットしない（`.env.example` を参考に各自設定）

## 開発メンバー

- リーダー: [あなたの名前]
- メンバー: 3-4名

## 開発期間

2026年2月 〜 2026年6月（4ヶ月）