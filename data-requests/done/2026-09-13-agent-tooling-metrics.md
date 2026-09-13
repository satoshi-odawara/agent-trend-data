---
type: 新規メトリクス
priority: 高
---

## 欲しいデータ

各リポジトリについて、「エージェント向け指示文書(CLAUDE.md/AGENTS.md)の
中身」だけでなく、「ツール化」の実態を示すフィールド:

- `has_skills_dir` (0/1): `.claude/skills/` 等、再利用可能なskills定義の
  ディレクトリ/仕組みの有無
- `skills_count` (integer): 上記が存在する場合のskills定義数
- `has_custom_commands` (0/1): `.claude/commands/` 等、カスタムslash
  commandsの有無
- `custom_commands_count` (integer)
- `has_hooks_config` (0/1): `.claude/settings.json` 等でのhooks設定
  (pre-commit/PostToolUse等、エージェントの挙動を機械的に制約する仕組み)
  の有無
- `mcp_servers_count` (integer, 可能であれば): 設定されているMCPサーバー数

`has_agent_instructions`/`agent_doc_*` 系と対になる「ツール化側」の指標
として、既存フィールドと同じ粒度(0/1 or count)で追加してほしい。

## 根拠

agent-trend-playbookの最初の検証テーマ「ルール化(CLAUDE.md等)とツール化
(skills等)の境界線」(`agent-trend-playbook/candidates/2026-09-13-rule-vs-tool-boundary.md`)
で、「指示文書の量・内容」と「ツール化の程度」を突き合わせて比較する
計画だが、現行の `metrics.json` / `schema/SCHEMA.md`(v1, 2026-09-13)には
指示文書側の指標(`agent_doc_char_count` 等)しかなく、ツール化側の実態を
示すフィールドが存在しない。このため対象20リポジトリ全体を横断した定量
比較ができず、少数サンプルの手動読解のみに頼らざるを得ない状態になって
いる。

サーベイを始めたばかりの初期段階の要望であるため、今後同種のテーマを
扱う際にも再利用できるよう、早めの収集開始を希望する。

## Issue化する場合の下書き(agent-trend-radar Issue.md 形式)

```
## #N リポジトリのツール化指標(skills/commands/hooks)を収集する

**概要**: 各リポジトリについて、エージェント向けskills定義・カスタム
slash commands・hooks設定の有無/数を収集し、metrics.jsonに追加する。

**変更対象ファイル**
- schema/SCHEMA.md(フィールド定義追記)
- metrics.json 生成ロジック(収集システム側、新規フィールド追加)

**タスク**
- [ ] `.claude/skills/`(または類似ディレクトリ)の有無・数を検出する
- [ ] `.claude/commands/`(または類似)の有無・数を検出する
- [ ] hooks設定(`.claude/settings.json` 内の `hooks` 等)の有無を検出する
- [ ] MCPサーバー設定数を検出する(可能であれば)
- [ ] schema/SCHEMA.md にフィールド定義を追記する

**完了条件**: 対象リポジトリ分の metrics.json に上記フィールドが追加され、
SCHEMA.md に定義が反映されている。

**依存**: なし
```

このリポジトリ(agent-trend-data)からagent-trend-radarへの直接のIssue化は
行っていません。上記は起票時にそのまま使えるよう下書きとして用意した
ものです。採否・実際のIssue化はご判断ください。
