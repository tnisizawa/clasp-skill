# ログ取得（clasp logs）

GAS の `console.log` / `console.error` の出力を CLI から取得できる。

## 前提条件

1. **GCP プロジェクトとの紐づけ** が必要（デフォルトの自動管理プロジェクトでは不可）
2. GCP で **Cloud Logging API** が有効であること
3. `.clasp.json` に `"projectId"` が設定されていること

## セットアップ手順

### 1. GCP プロジェクト作成
[GCP コンソール](https://console.cloud.google.com/projectcreate) で新規プロジェクトを作成（例: `my-gas-dev`）

### 2. GAS プロジェクトに紐づけ
Apps Script エディタ → プロジェクトの設定（歯車）→ GCP プロジェクト → 「プロジェクトを変更」→ GCP のプロジェクト番号を入力

**⚠️ 注意:** GCP プロジェクトを変更すると既存のユーザー認証が取り消される。**本番環境には適用しない**こと。開発用プロジェクトにのみ設定する。

### 3. Cloud Logging API の有効化
GCP コンソール → 左上メニュー（≡）→ API とサービス → 「Cloud Logging API」を検索して有効化

### 4. `.clasp.json` に projectId を追加

```json
{
  "scriptId": "...",
  "projectId": "my-project-name-123456",
  "rootDir": "./"
}
```

※ `projectId` には GCP プロジェクトID（文字列: `my-project-name-123456` 等）を指定する。プロジェクト番号（数字のみ）ではないので注意。GCP コンソール → ダッシュボード → プロジェクト情報で確認可能。

## コマンド

```bash
# 最新ログを取得
clasp logs

# リアルタイム監視
clasp logs --watch

# タイムスタンプなしで表示
clasp logs --simplified
```

## 注意事項

| 項目 | 説明 |
|------|------|
| 開発用のみ | 本番（dist）の GCP プロジェクトを変更するとユーザーの再認証が必要になる |
| `console.log` / `console.error` を優先 | `clasp logs` は Cloud Logging 前提。`Logger.log` は実行ログ画面など環境差があるため、CLI で追うログは `console.log` / `console.error` に寄せる |
| ログの遅延 | Cloud Logging への反映には数秒〜十数秒の遅延がある場合がある |
