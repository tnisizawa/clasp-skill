# 初回セットアップ（インストール〜動作確認）

clasp を初めて使うマシンでの準備手順。すでに `clasp login` 済みなら不要。

## 1. clasp のインストール

Node.js が必要（**v22.0.0 以上**。clasp 3.x 公式 README 準拠）。

```bash
node --version   # v22 以上であることを確認
npm install -g @google/clasp
clasp --version  # 例: 3.1.3
```

Node が古い場合は先に Node.js を更新する（nvm / インストーラ等、環境の流儀に合わせる）。

## 2. Google アカウントでログイン

```bash
clasp login
```

- ブラウザが開くので、GAS を使う Google アカウントで認証する
- 成功すると `~/.clasprc.json` に認証情報が保存される（**このファイルは秘密情報。git にコミットしない**）
- 別アカウントに切り替えたいときは `clasp logout` → `clasp login`

## 3. Apps Script API の有効化（つまずきポイント）

clasp からプロジェクトを操作するには、アカウントごとに Apps Script API を ON にする必要がある。

1. https://script.google.com/home/usersettings を開く
2. 「Google Apps Script API」を **ON** にする

これを忘れると、`clasp create` / `clasp clone` / `clasp push` などで次のエラーが出る:

```
User has not enabled the Apps Script API. Enable it by visiting
https://script.google.com/home/usersettings then retry.
```

→ 上記ページで ON にしてから、**数分待って**再実行する（反映に時間がかかることがある）。
  成功判定は手順4の `clasp list-scripts` が通ること。

## 4. 動作確認

```bash
clasp --version       # バージョンが表示されればインストール OK
clasp list-scripts    # 自分の GAS プロジェクト一覧が出ればログイン+API 有効化 OK
# 旧 alias: clasp list
```

`clasp list-scripts` が成功すれば準備完了。次は用途に応じて:

- 新規プロジェクトの接続・作成 → `references/init.md`
- コードのプッシュ → SKILL.md「push フロー」節
