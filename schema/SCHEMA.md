# SCHEMA

`snapshots/YYYY-MM-DD/metrics.json` および `latest/metrics.json` のフィールド定義です。
このリポジトリにおけるスキーマの正はこのファイルです。収集システム側の実装が
この定義と食い違っている場合は、収集システム側を直してください。

## 運用ルール

フィールドを追加・変更する際は、このファイルの更新をセットで行ってください。
スキーマ変更を伴わないデータ更新は通常運用として許容されますが、フィールドの
追加・削除・型変更・意味の変更は必ずこのファイルに反映してください。

- **バージョン**: v1(2026-09-13 初回収集分に基づき定義)

## `metrics.json`

トップレベル構造:

| フィールド | 型 | 説明 |
| --- | --- | --- |
| `generated_at` | string (ISO 8601, UTC) | このスナップショットの生成日時 |
| `repos` | array<object> | 収集対象リポジトリごとのメトリクス。各要素は下記「repos の各要素」を参照 |

## `repos` の各要素

対象は1つのGitHubリポジトリ。主に、リポジトリが「エージェント向けの指示書
(AGENTS.md / CLAUDE.md 等)」をどの程度整備しているかを表す構造化メトリクス。

| フィールド | 型 | 説明 |
| --- | --- | --- |
| `repo` | string | `owner/repo` 形式のGitHubリポジトリ識別子(例: `remix-run/remix`) |
| `segment` | string (enum) | リポジトリの分類。`adopter`(エージェント技術を利用する側のOSSプロジェクト) / `tool`(エージェント関連のツール・フレームワークを提供する側) |
| `monetization_model` | string (enum) | リポジトリの収益化・運営モデル。`big_corp_internal` / `commercial_saas` / `individual_community` / `nonprofit_foundation` |
| `has_agent_instructions` | 0 or 1 | エージェント向け指示ファイル(AGENTS.md, CLAUDE.md 等)が存在するか |
| `has_tests` | 0 or 1 | テストコード・テストディレクトリの有無 |
| `has_eval` | 0 or 1 | eval(評価)の仕組みの有無 |
| `has_ci` | 0 or 1 | CI設定の有無 |
| `has_security_policy` | 0 or 1 | セキュリティポリシー(SECURITY.md 等)の有無 |
| `agent_doc_char_count` | integer | エージェント向け指示ファイルの文字数。`has_agent_instructions` が 0 の場合は 0 |
| `agent_doc_heading_count` | integer | エージェント向け指示ファイル内の見出し数 |
| `agent_doc_has_code_block` | 0 or 1 | エージェント向け指示ファイルにコードブロックを含むか |
| `agent_doc_mentions_test` | 0 or 1 | テストへの言及があるか |
| `agent_doc_mentions_lint` | 0 or 1 | lintへの言及があるか |
| `agent_doc_mentions_security` | 0 or 1 | セキュリティへの言及があるか |
| `agent_doc_mentions_commit_convention` | 0 or 1 | コミット規約への言及があるか |
| `agent_doc_mentions_tool_usage` | 0 or 1 | ツールの使い方への言及があるか |
| `agent_doc_mentions_repo_structure` | 0 or 1 | リポジトリ構成への言及があるか |
| `agent_doc_mentions_boundaries` | 0 or 1 | やって良い事/やってはいけない事(境界線)への言及があるか |
| `agent_doc_mentions_pr_review` | 0 or 1 | PRレビューへの言及があるか |
| `agent_doc_mentions_release_process` | 0 or 1 | リリースプロセスへの言及があるか |

補足:

- `has_agent_instructions` が 0 のとき、`agent_doc_` プレフィックスを持つ全フィールドは 0 になる(指示ファイルが存在しないため)。
- 0/1 の各フィールドは真偽値をintで表現したもの。

## `manifest.json`

| フィールド | 型 | 説明 |
| --- | --- | --- |
| `dates` | array<string (YYYY-MM-DD)> | `snapshots/` 配下に存在するスナップショット日付の一覧 |
