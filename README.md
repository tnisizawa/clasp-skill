# clasp-skill

An Agent Skill that teaches Claude Code / Codex how to operate [clasp](https://github.com/google/clasp) (the Google Apps Script CLI) safely. Just say "clasp push" to your agent, and it follows safe procedures — including an accident-prevention decision flow.

*日本語の説明は[後半](#clasp-skill-日本語)にあります。*

## Features

- Accident prevention when connecting existing projects (a decision flow that keeps `clasp clone` from wiping your local code)
- Deployment-type detection (web app vs library) with safe update procedures for each
- Versioned deployments and rollback procedures
- Multi-environment push (development / distribution)
- Workarounds for Windows / Git Bash quirks (e.g. clasp output not being captured)
- Guided first-time setup (installation through enabling the Apps Script API)

## Prerequisites

- Node.js v22+, clasp, and a Google account — setup instructions are included in `references/setup.md`. After installing the skill, you can simply ask your agent to "set up clasp".

## Installation

```bash
# For Claude Code
git clone https://github.com/tnisizawa/clasp-skill ~/.claude/skills/clasp

# For Codex
git clone https://github.com/tnisizawa/clasp-skill ~/.codex/skills/clasp

# To use with both, clone to both locations (or use a symlink)
```

## Usage

Once cloned, just talk to your agent in plain language:

- "Set up clasp" (first-time install, login, API activation)
- "Connect this GAS project with clasp" (runs the accident-prevention flow)
- "clasp push"
- "Deploy the web app" / "Roll back to the previous version"
- "Pull in the changes I made in the web editor"

## Updating

```bash
git -C ~/.claude/skills/clasp pull
```

## Notes

- The skill documentation (`SKILL.md` and `references/`) is written in Japanese. Claude Code / Codex understand it natively, so the skill works fine in English-language sessions too. An English translation of the skill entry point is available in [`SKILL.en.md`](SKILL.en.md).
- This skill makes no network requests of its own — it only drives the `clasp` CLI and Google's official auth flow.

## License

MIT License — see [LICENSE](LICENSE).

---

<a id="clasp-skill-日本語"></a>

# clasp-skill（日本語）

Google Apps Script の CLI ツール [clasp](https://github.com/google/clasp) の運用ノウハウを、Claude Code / Codex に教える Agent Skill です。「clasp push して」と話しかけるだけで、エージェントが安全な手順（事故防止の判定フロー込み）で作業してくれるようになります。

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
