# AGENTS.md — SAKURA OS

> このファイルは SAKURA OS プロジェクト専用のルールを定義する。
> 全体共通ルールは `/Users/sakura/Desktop/Development/AGENTS.md` を参照。

---

## プロジェクト概要

- **バージョン**: v1.8.1
- **本番URL**: `https://sakuranet-co.jp/system/sakura-os.html`
- **ファイルパス**: `CTO/SAKURA-OS/sakura-os.html`（単一ファイル構成）

---

## Codex への指示

### やること
- JavaScript ロジックの実装・リファクタリング
- 機能追加の実装（設計は Claude Code が決定してから依頼）
- コードコメントの整備

### やらないこと
- HTML/CSS の直接変更（デザインは DESIGN.md 準拠・Claude Code が担当）
- データ削除・localStorage のクリア
- 本番 URL へのデプロイ

---

## 技術仕様

- **構成**: 単一 HTML ファイル（HTML + CSS + JavaScript 一体型）
- **データ保存**: localStorage
- **マスコット**: 花帆ちゃん（kahochan）— 変更・削除禁止
- **SAKURA AI 連携**: `/api/chat` エンドポイント（顧客検索 Phase2）

---

## 重要な注意事項

- localStorage のデータ構造を変更すると既存ユーザーデータが消える → **必ずマイグレーション処理を入れる**
- 単一ファイルのため、変更前に必ず `backups/` にコピーを取る
- 花帆ちゃんのアニメーション・表示ロジックは変更禁止

---

*Last updated: 2026-05-23*
