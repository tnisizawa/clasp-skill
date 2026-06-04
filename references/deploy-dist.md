# deploy-dist.sh（マルチ環境 push）

開発用/リリース用など複数の GAS プロジェクトがある場合に、`.clasp.dist.json` を一時的に `.clasp.json` にスワップして、配布版 project へ push するスクリプト。

**重要:** このテンプレートは `clasp push` までを行う。ウェブアプリやライブラリの公開 URL には反映されない。push 完了後、`.clasp.json` は元に戻る。配布版の公開 URL に反映する場合は、GAS エディタで操作するか、dist 設定を適用した状態で `references/deploy.md` のバージョン付きデプロイを実行する。

## テンプレート

### 基本版（ライブラリ等、.clasp.json のスワップのみ）

```bash
#!/bin/bash
set -euo pipefail
# 配布版 project へ push
readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

readonly BAK_FILE="$SCRIPT_DIR/.clasp.json.bak.$(date +%Y%m%d%H%M%S)"

copy_file() {
  node -e "const fs=require('fs');fs.copyFileSync(process.argv[1],process.argv[2])" "$1" "$2"
}

restore_file() {
  [ -f "$1" ] || return 0
  node -e "const fs=require('fs');fs.copyFileSync(process.argv[1],process.argv[2]);fs.unlinkSync(process.argv[1])" "$1" "$2"
}

cleanup() {
  restore_file "$BAK_FILE" "$SCRIPT_DIR/.clasp.json" || {
    echo "Error: failed to restore .clasp.json from $BAK_FILE" >&2
    return 1
  }
}
trap cleanup EXIT

[ -f "$SCRIPT_DIR/.clasp.dist.json" ] || { echo "Error: .clasp.dist.json not found in $SCRIPT_DIR" >&2; exit 1; }
[ -f "$SCRIPT_DIR/.clasp.json" ] || { echo "Error: .clasp.json not found in $SCRIPT_DIR" >&2; exit 1; }
echo "=== 配布版 project へ push します ==="
echo "scriptId: $(grep scriptId "$SCRIPT_DIR/.clasp.dist.json")"
if [[ "${DRY_RUN:-}" = "1" ]]; then
  echo "DRY_RUN=1: .clasp.dist.json を確認しました。clasp push は実行しません"
  exit 0
fi
if [[ "${SKIP_CONFIRM:-}" = "1" ]]; then
  echo "SKIP_CONFIRM=1: 確認をスキップします"
elif ! [[ -t 0 ]]; then
  echo "非対話モードです。続行するには SKIP_CONFIRM=1 を設定してください" >&2
  exit 1
else
  read -p "続行しますか? (y/N) " confirm
  [[ "$confirm" != "y" && "$confirm" != "Y" ]] && echo "キャンセルしました" && exit 0
fi

copy_file "$SCRIPT_DIR/.clasp.json" "$BAK_FILE"
copy_file "$SCRIPT_DIR/.clasp.dist.json" "$SCRIPT_DIR/.clasp.json"

# Git Bashではclaspの出力がキャプチャされない問題があるため、Node.js経由で実行
if cd "$SCRIPT_DIR" && node -e "const{execSync}=require('child_process');try{execSync('clasp push --force',{stdio:'inherit'})}catch(e){process.exit(e.status||1)}"; then
  echo "✅ 配布版 project への push 完了"
  echo ".clasp.json は元に戻ります。公開 URL に反映する場合は、GAS エディタまたは dist 設定を適用した状態でバージョン付きデプロイしてください"
  # 必要なら、ここに echo "📝 配布版スプレッドシートを日付付きでリネームしてね" を追加する
else
  echo "❌ clasp push 失敗。配布版 project への push は中断されました" >&2
  exit 1
fi
```

### 拡張版（appsscript.json のスワップも必要な場合）

バインドスクリプトで開発用/リリース用でライブラリ参照先が異なる場合など、`appsscript.json` もスワップが必要な場合:

```bash
#!/bin/bash
set -euo pipefail
# 配布版バインドスクリプトへ push
readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

readonly BAK_CLASP="$SCRIPT_DIR/.clasp.json.bak.$(date +%Y%m%d%H%M%S)"
readonly BAK_APPS="$SCRIPT_DIR/appsscript.json.bak.$(date +%Y%m%d%H%M%S)"

copy_file() {
  node -e "const fs=require('fs');fs.copyFileSync(process.argv[1],process.argv[2])" "$1" "$2"
}

restore_file() {
  [ -f "$1" ] || return 0
  node -e "const fs=require('fs');fs.copyFileSync(process.argv[1],process.argv[2]);fs.unlinkSync(process.argv[1])" "$1" "$2"
}

cleanup() {
  restore_file "$BAK_CLASP" "$SCRIPT_DIR/.clasp.json" || {
    echo "Error: failed to restore .clasp.json from $BAK_CLASP" >&2
    return 1
  }
  restore_file "$BAK_APPS" "$SCRIPT_DIR/appsscript.json" || {
    echo "Error: failed to restore appsscript.json from $BAK_APPS" >&2
    return 1
  }
}
trap cleanup EXIT

[ -f "$SCRIPT_DIR/.clasp.dist.json" ] || { echo "Error: .clasp.dist.json not found in $SCRIPT_DIR" >&2; exit 1; }
[ -f "$SCRIPT_DIR/appsscript.dist.json" ] || { echo "Error: appsscript.dist.json not found in $SCRIPT_DIR" >&2; exit 1; }
[ -f "$SCRIPT_DIR/.clasp.json" ] || { echo "Error: .clasp.json not found in $SCRIPT_DIR" >&2; exit 1; }
[ -f "$SCRIPT_DIR/appsscript.json" ] || { echo "Error: appsscript.json not found in $SCRIPT_DIR" >&2; exit 1; }
echo "=== 配布版バインドスクリプトへ push します ==="
echo "scriptId: $(grep scriptId "$SCRIPT_DIR/.clasp.dist.json")"
if [[ "${DRY_RUN:-}" = "1" ]]; then
  echo "DRY_RUN=1: .clasp.dist.json と appsscript.dist.json を確認しました。clasp push は実行しません"
  exit 0
fi
if [[ "${SKIP_CONFIRM:-}" = "1" ]]; then
  echo "SKIP_CONFIRM=1: 確認をスキップします"
elif ! [[ -t 0 ]]; then
  echo "非対話モードです。続行するには SKIP_CONFIRM=1 を設定してください" >&2
  exit 1
else
  read -p "続行しますか? (y/N) " confirm
  [[ "$confirm" != "y" && "$confirm" != "Y" ]] && echo "キャンセルしました" && exit 0
fi

copy_file "$SCRIPT_DIR/.clasp.json" "$BAK_CLASP"
copy_file "$SCRIPT_DIR/.clasp.dist.json" "$SCRIPT_DIR/.clasp.json"
copy_file "$SCRIPT_DIR/appsscript.json" "$BAK_APPS"
copy_file "$SCRIPT_DIR/appsscript.dist.json" "$SCRIPT_DIR/appsscript.json"

# Git Bashではclaspの出力がキャプチャされない問題があるため、Node.js経由で実行
if cd "$SCRIPT_DIR" && node -e "const{execSync}=require('child_process');try{execSync('clasp push --force',{stdio:'inherit'})}catch(e){process.exit(e.status||1)}"; then
  echo "✅ 配布版バインドスクリプトへの push 完了"
  echo ".clasp.json は元に戻ります。公開 URL に反映する場合は、GAS エディタまたは dist 設定を適用した状態でバージョン付きデプロイしてください"
  # 必要なら、ここに echo "📝 配布版スプレッドシートを日付付きでリネームしてね" を追加する
else
  echo "❌ clasp push 失敗。配布版バインドスクリプトへの push は中断されました" >&2
  exit 1
fi
```

## 重要なポイント

| ポイント | 説明 |
|----------|------|
| `set -euo pipefail` | **必須。** エラー即停止 + 未定義変数検出 + パイプ失敗検出。これがないと `clasp push` が失敗しても「push 完了」と表示される |
| `readonly SCRIPT_DIR` | スクリプトの場所を不変で保持。`dirname` 解決失敗時は即終了 |
| `trap cleanup EXIT` | 正常終了・エラー終了・Ctrl+C いずれの場合でも `.bak` から元に戻す |
| `DRY_RUN=1` | 対象設定ファイルの存在と push 先を確認し、`clasp push` は実行しない |
| `clasp push` のエラーチェック | Node.js 経由の `clasp push --force` で成功/失敗を判定し、失敗時は明示的にエラーメッセージを表示して `exit 1` |
| 確認プロンプト | `read -p` で push 先を表示して確認。誤 push 防止 |
| リネームリマインダー | バインド先スプレッドシートの誤上書き防止やバージョン識別が必要な運用では、push 完了後に日付付きリネームを促す `echo` をプロジェクト固有で追加する |

## .gitignore に追加

`*.bak` ファイルが git に入らないようにする:

```
*.bak*
```
