# Web エディタの変更を取り込む（clasp pull の日常運用）

**棲み分け**: 初回接続（ローカルとクラウドを初めてつなぐ）は `references/init.md` を使う。
このファイルは**接続済みプロジェクト**で、Web エディタ側の編集をローカルへ取り込む日常同期の手順。
「clasp pull は全ファイル上書き」の詳細な背景は init.md ケースA を出典とする。

## 大前提: clasp pull は全ファイル上書き

`clasp pull` はクラウド側の内容で**ローカルの対象ファイルをすべて上書き**する。
マージはしない。ローカルにしかない変更は消える。だから git が頼みの綱になる。

## 手順

```bash
# 1. git が clean であることを確認（未コミット変更があるなら先にコミット or stash）
git status --porcelain
# → 出力が空であること。空でなければ:
#    git stash --include-untracked  （推奨: 未追跡ファイルも含めて一時退避）
#    または git add -A && git commit -m "WIP: pull 前の退避"
#      ※ git add -A は .clasprc.json などの秘密情報を誤ってコミットするリスクがある。
#         差分を git status / git diff で確認したうえで選択的に add すること。

# 2. クラウド側の変更を取り込む
clasp pull

# 3. 何が変わったかを git で確認
git diff
git status

# 4. 意図しない上書きがあれば、ファイル単位で元に戻す
git checkout -- <file>
```

## ローカル・クラウド両方に変更がある場合

順序が重要。**先に pull、diff で統合、最後に push**。

1. ローカルの変更をコミットしておく（退避点を作る）
2. `clasp pull` → ローカルがクラウド版で上書きされる
3. `git diff HEAD` でクラウド側の変更点を確認
4. 必要なローカル変更を `git checkout -p` や手編集で復元・統合する
5. 統合結果を `clasp push` でクラウドへ反映（push 手順は SKILL.md「push フロー」節）

## やってはいけないこと

- git が dirty なまま `clasp pull` → ローカルの未保存変更が消えても復元できない
- pull と push の順序を逆にする → クラウド側の編集（Web エディタでの修正）を push で潰す
