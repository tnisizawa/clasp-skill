# プロジェクト初期化

新規に GAS プロジェクトをローカル/クラウドで接続するときの手順。

**棲み分け**: このファイルは**初回接続**用。接続済みプロジェクトで Web エディタ側の編集を取り込む日常同期は `references/pull.md` を使う。

## 判定フロー（必ずこの順序で確認する）

```
Q1: クラウド側に既存のGASプロジェクトがあるか？
├── YES → Q2: ローカルにコード（.jsファイル等）があるか？
│   ├── YES → ケースA: 手動リンク（最も注意が必要）
│   └── NO  → ケースB: clasp clone
└── NO  → Q3: 既存スプレッドシート等に紐づけるか？
    ├── YES → ケースC: clasp create --parentId
    └── NO  → ケースD: clasp create --type
```

## ケースA: ローカルにコードあり + クラウドに既存プロジェクトあり

**最もトラブルが起きやすいケース。** ローカルのコードを正として、クラウドに反映する。

**⚠️ 絶対にやってはいけないこと:**
- `clasp clone` → **ローカルファイルがクラウドで上書きされ、ローカルのコードが消える**
- `clasp create` → **新規プロジェクトが作成され、既存プロジェクトとは別物になる**

**手順:**

```bash
# exit / trap の作用範囲を閉じるため、ケースAは一括で実行する
bash <<'SCRIPT_EOF'
set -euo pipefail

# Step 1: gitでバックアップ確認（未コミットの変更は必ずコミットまたはstash）
# clasp pull はローカルファイルを上書きするため、clean な状態が必須
[[ -z "$(git status --porcelain)" ]] || {
  echo "未コミットの変更または未追跡ファイルがあります。先にコミットまたは git stash --include-untracked してください"
  exit 1
}

# Step 2: .clasp.json を手動作成（scriptIdを指定するだけ）
# ※ clasp clone や clasp create は使わない
cat > .clasp.json << 'JSON_EOF'
{
  "scriptId": "スクリプトID",
  "rootDir": "./"
}
JSON_EOF

# Step 3: appsscript.json をクラウドから取得（安全な方法）
# clasp pull は全ファイルを上書き＋新規ファイルを追加するため、
# 一時ディレクトリで pull し appsscript.json だけをコピーする
PULL_DIR=$(mktemp -d)
cleanup() {
  rm -rf "$PULL_DIR"
}
trap cleanup EXIT
cp .clasp.json "$PULL_DIR/"
(cd "$PULL_DIR" && clasp pull)
# rootDir 設定時は appsscript.json がサブディレクトリに配置される場合がある
PULLED_JSON=$(find "$PULL_DIR" -name appsscript.json | head -n 1)
[ -n "$PULLED_JSON" ] || { echo "Error: appsscript.json not found after clasp pull" >&2; exit 1; }
# rootDir 設定がある場合はそのディレクトリにコピー（node は clasp の前提）
ROOT_DIR=$(node -p "(()=>{try{const path=require('path');const r=require('./.clasp.json').rootDir;return (r&&!path.isAbsolute(r))?r:'.'}catch(e){return '.'}})()")
mkdir -p "$ROOT_DIR"
cp "$PULLED_JSON" "$ROOT_DIR/appsscript.json"

# Step 4: appsscript.json を調整（SKILL.md「push 前の準備」テンプレート参照）
# - timeZone を "Asia/Tokyo" に
# - oauthScopes を必要に応じて追加
SCRIPT_EOF
```

appsscript.json の調整が終わったら、ローカルのコードをクラウドに反映する。

```bash
node -e "const{execSync}=require('child_process');try{execSync('clasp push --force',{stdio:'inherit'})}catch(e){process.exit(e.status||1)}"
```

**Step 3 の代替手段:** `clasp pull` を使わず、SKILL.md の `appsscript.json` テンプレートをそのまま配置してもよい。

## ケースB: ローカルにコードなし + クラウドに既存プロジェクトあり

クラウドのコードをローカルにダウンロードする。

```bash
# ディレクトリを作成して移動
mkdir my-gas-project && cd my-gas-project

# クラウドからクローン（.clasp.json + 全ファイルがダウンロードされる）
clasp clone "スクリプトID"
```

## ケースC: クラウドにプロジェクトなし + 既存スプレッドシート等に紐づける

```bash
# ⚠️ --type sheets をつけてはいけない（新しいスプレッドシートが作成されてしまう）
clasp create --parentId "スプレッドシートID" --title "プロジェクト名"
```

**特徴:** `SpreadsheetApp.getActiveSpreadsheet()`, `onOpen()` 等が使える。

## ケースD: クラウドにプロジェクトなし + スタンドアロン or 新規シート

```bash
# スタンドアロン
clasp create --type standalone --title "プロジェクト名"

# 新規スプレッドシート + スクリプトを同時作成
clasp create --type sheets --title "プロジェクト名"
```

## ケースC・Dの共通後続手順

1. `appsscript.json` を SKILL.md のテンプレートに合わせて配置（`timeZone: "Asia/Tokyo"`、必要な `oauthScopes`）
2. コードを作成後、SKILL.md の手順で `clasp push --force`

## ID の取得方法

| 種類 | URL内の位置 |
|------|------------|
| スクリプトID | `script.google.com/d/{スクリプトID}/edit` または GAS エディタ → プロジェクトの設定 |
| スプレッドシートID (parentId) | `docs.google.com/spreadsheets/d/{parentId}/edit` |
| ドキュメントID (parentId) | `docs.google.com/document/d/{parentId}/edit` |

## トラブルシューティング

### 重複プロジェクトができた場合

1. `https://script.google.com/home` → 不要なプロジェクト → 三点メニュー → 削除
2. または: スプレッドシート → 「拡張機能 → Apps Script」→ プロジェクトの設定 → 一番下の「プロジェクトを削除」

### clasp login が必要な場合

```bash
clasp login
```
