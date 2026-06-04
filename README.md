# clasp-skill

Google Apps Script の CLI ツール [clasp](https://github.com/google/clasp) の運用ノウハウを、Claude Code / Codex に教える Agent Skill です。「clasp push して」と話しかけるだけで、エージェントが安全な手順（事故防止の判定フロー込み）で作業してくれるようになります。

**English**: An Agent Skill that teaches Claude Code / Codex how to operate [clasp](https://github.com/google/clasp) (the Apps Script CLI) safely — including an accident-prevention decision flow for connecting existing projects, multi-environment push, deployment-type detection (web app vs library), version rollback, and Windows/Git Bash quirks. Documentation is in Japanese.

## 特徴

- 既存プロジェクト接続時の事故防止（`clasp clone` でローカルコードを消さないための判定フロー）
- デプロイ種別（ウェブアプリ / ライブラリ）の判定と、それぞれの安全な更新手順
- バージョン付きデプロイとロールバック手順
- 開発版 / 配布版のマルチ環境 push
- Windows / Git Bash 特有の問題（clasp の出力がキャプチャされない等）への対処
- 初回セットアップ（インストール〜Apps Script API 有効化）のガイド付き

## 前提

- Node.js v22 以上と clasp、Google アカウント（セットアップ手順は `references/setup.md` に含まれています。インストール後にエージェントへ「clasp のセットアップして」と頼んでも OK）

## インストール

```bash
# Claude Code で使う
git clone https://github.com/tnisizawa/clasp-skill ~/.claude/skills/clasp

# Codex で使う
git clone https://github.com/tnisizawa/clasp-skill ~/.codex/skills/clasp

# 両方で使う場合は2か所に clone（またはシンボリックリンク）
```

## 使い方

clone したら、エージェントにふだんの言葉で話しかけるだけです。

- 「clasp のセットアップして」（初回インストール〜ログイン〜API 有効化）
- 「この GAS プロジェクトを clasp で接続して」（事故防止の判定フローが走ります）
- 「clasp push して」
- 「ウェブアプリをデプロイして」「ひとつ前のバージョンに戻して」
- 「Web エディタで直した分を取り込んで」

## 更新

```bash
git -C ~/.claude/skills/clasp pull
```

## ライセンス

MIT License（[LICENSE](LICENSE) を参照）
