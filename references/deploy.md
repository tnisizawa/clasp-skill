# バージョン付きデプロイ（deploy）

`clasp push` は HEAD（開発用）にコードを反映するだけ。公開 URL に反映するにはバージョン付きデプロイが必要。

## 最重要: デプロイ種別による手順の違い

既存デプロイに新しいバージョンを反映する更新は CLI で行える。初回ウェブアプリ作成、実行ユーザー、アクセス権限の変更は GAS UI で行う。ウェブアプリは更新後に既存 URL を開いて必ず動作確認する。

| デプロイ種別 | CLI で更新可能？ | 備考 |
|-------------|-------------------------|------|
| **ライブラリ** | ✅ 可能 | `clasp deploy -i` で安全に更新できる |
| **ウェブアプリ** | ✅ 可能 | 既存デプロイは `clasp update-deployment` または `clasp deploy -i` で更新。初回作成・権限変更は GAS UI。更新後確認必須 |

## デプロイ前の確認フロー（必須）

デプロイを依頼された時は、**必ず以下を確認してから手順を選ぶ:**

1. **doGet / doPost 関数があるか？** → あればウェブアプリの可能性が高い
2. **ユーザーに確認**: 「このプロジェクトはウェブアプリですか？ライブラリですか？」
3. 種別に応じて以下の手順を選ぶ

## ライブラリのデプロイ

CLI で完結できる。

```bash
# 1. コードをプッシュ（SKILL.md の手順を参照: Git Bash では Node.js 経由で実行）
node -e "const{execSync}=require('child_process');try{execSync('clasp push --force',{stdio:'inherit'})}catch(e){process.exit(e.status||1)}"

# 2. デプロイ一覧を確認
clasp list-deployments
# 旧 alias: clasp deployments

# 3. 既存デプロイメントを更新（URLを維持）
clasp update-deployment "デプロイメントID" -d "変更内容"
# 旧 alias: clasp deploy -i "デプロイメントID" -d "変更内容"

# または新規デプロイ
clasp deploy -d "説明"
```

## ウェブアプリのデプロイ

既存のウェブアプリ URL に新しいコードバージョンを反映するだけなら CLI で更新できる。実行ユーザー・アクセス権限などのウェブアプリ設定を変える場合は GAS UI から操作する。

### 手順

```bash
# 1. コードをプッシュ（SKILL.md の手順と同じく Node.js 経由）
node -e "const{execSync}=require('child_process');try{execSync('clasp push --force',{stdio:'inherit'})}catch(e){process.exit(e.status||1)}"

# 2. デプロイ一覧を確認
clasp list-deployments

# 3. 既存ウェブアプリデプロイを更新（URLを維持）
clasp update-deployment "デプロイメントID" -d "変更内容"

# 4. 既存の公開 URL を開いて動作確認
```

`update-deployment` が使えない古い clasp では、alias として `clasp deploy -i "デプロイメントID" -d "変更内容"` を使う。
`clasp` 3.1.3 では、バージョン番号を省略すると新バージョンを作成してから指定 deploymentId を更新する。

### 初回ウェブアプリデプロイ

1. **「デプロイ」→「新しいデプロイ」**
2. 歯車アイコン → **「ウェブアプリ」** を選択
3. 設定（公開範囲は用途に応じて確認する）:
   - **次のユーザーとして実行**: 通常は「自分」。プロジェクトの運用方針に合わせる
   - **アクセスできるユーザー**: 「自分のみ」「ドメイン内」「全員」などから選ぶ。外部公開が必要な場合だけ「全員」にする
4. **「デプロイ」** をクリック

## ロールバック（旧バージョンへ戻す）

デプロイ後に問題が見つかったとき、既存の URL を維持したまま旧バージョンへ戻せる。

```bash
# 1. バージョン一覧を確認（戻し先のバージョン番号を決める）
clasp list-versions
# 旧 alias: clasp versions

# 2. デプロイ一覧から対象のデプロイメントIDを確認
clasp list-deployments

# 3. 既存デプロイメントを旧バージョンに向け直す（URLは変わらない）
clasp deploy -V <旧バージョン番号> -i "デプロイメントID" -d "rollback to vN"
```

- `-V`（`--versionNumber`）と `-i`（`--deploymentId`）の組み合わせで「既存デプロイを指定バージョンへ更新」になる（clasp 3.1.3 で確認）
- `-i` を忘れると**新規デプロイが作られて URL が変わる**ので注意
- ウェブアプリの場合、戻した後も**公開 URL を開いて動作確認必須**（更新時と同じ運用ルール）
- 戻し先のバージョンが存在しない（バージョン未作成のままデプロイしていた）場合は、`clasp create-version`（旧 alias: `clasp version`）で現状を固めてから運用を見直す

## 不要なデプロイの削除

```bash
clasp undeploy "デプロイメントID"
```

**注意:** `@HEAD` は削除できない。

## 注意事項

| 項目 | 説明 |
|------|------|
| push だけでは反映されない | 公開URLに反映するには push 後にデプロイが必要 |
| 初回ウェブアプリ作成はGUI | 実行ユーザー・アクセスできるユーザーを設定するため GAS UI で作成する |
| `-i` / deploymentId を忘れると新規デプロイになる | 既存 URL を維持したい場合は必ず既存デプロイメントIDを指定する |
| `@HEAD` は開発用 | `@HEAD` の URL（`/dev` で終わる）はテスト専用。本番には使わない |
| 権限変更はGUI | 「実行ユーザー」「アクセスできるユーザー」の変更はApps Scriptエディタからのみ可能 |
