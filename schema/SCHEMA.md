# SCHEMA

`snapshots/YYYY-MM-DD/metrics.json` および `latest/metrics.json` のフィールド定義です。
このリポジトリにおけるスキーマの正はこのファイルです。収集システム側の実装が
この定義と食い違っている場合は、収集システム側を直してください。

## 運用ルール

フィールドを追加・変更する際は、このファイルの更新をセットで行ってください。
スキーマ変更を伴わないデータ更新は通常運用として許容されますが、フィールドの
追加・削除・型変更・意味の変更は必ずこのファイルに反映してください。

- **バージョン**: v1.2(2026-09-14、`agent_doc_count`追加。v1.1は
  `has_skills_dir`〜`mcp_servers_count`の6フィールド追加、v1は
  2026-09-13初回収集分に基づき定義)
- **同日再収集時のsnapshots不変性**: `snapshots/<date>/metrics.json`が
  既に存在する場合、収集システムは上書きせずスキップし警告を出す
  (`latest/metrics.json`のみ常に最新化される)。過去スナップショットの
  不変性を保証するための挙動(2026-09-14、agent-trend-radar Issue #25で
  確定・実装。同日2回の収集が実際に発生し、この挙動を確認済み)。

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
| `agent_doc_count` | integer | 指示ファイル(CLAUDE.md/AGENTS.md)が見つかったディレクトリ数(重複排除、リポジトリ内の任意の深さを対象) |
| `agent_doc_char_count` | integer | 代表文書1件分の文字数。ルート直下に指示ファイルがあればそれを使い、無ければ見つかった中で最も浅いディレクトリのものを使う(複数文書は合算しない)。`has_agent_instructions` が 0 の場合は 0 |
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
| `has_skills_dir` | 0 or 1 | 再利用可能なSkill定義(`.claude/skills/`等)の有無 |
| `skills_count` | integer | Skill定義の数(`.claude/skills/`直下の子要素数) |
| `has_custom_commands` | 0 or 1 | カスタムslash commands(`.claude/commands/`等)の有無 |
| `custom_commands_count` | integer | カスタムcommand定義の数(`.claude/commands/`配下の`.md`ファイル数) |
| `has_hooks_config` | 0 or 1 | `.claude/settings.json`の`hooks`キーが空でないか |
| `mcp_servers_count` | integer | `.mcp.json`の`mcpServers`に定義されたMCPサーバー数 |

補足:

- `has_agent_instructions` が 0 のとき、`agent_doc_` プレフィックスを持つ全フィールドは 0 になる(指示ファイルが存在しないため)。
- 0/1 の各フィールドは真偽値をintで表現したもの。
- `agent_doc_mentions_repo_structure`〜`agent_doc_mentions_release_process`の
  4フィールドはLLM分類(収集システム側で`claude -p`ヘッドレス実行を利用)
  による判定であり、他の全フィールド(ファイル存在確認・キーワード一致・
  JSON構造確認によるルールベース判定)と異なり、同一入力に対して実行の
  たびに結果が変わりうる(非決定的)ことが実データで確認されている
  (2026-09-13、同日2回の収集間で`agent_doc_mentions_boundaries`・
  `agent_doc_mentions_pr_review`が変化した事例あり)。この4フィールドを
  時系列で比較する際は、値の変化が実際の指示文書の変更ではなく分類の
  ゆらぎに起因する可能性を考慮すること。
- `agent_doc_char_count`・`agent_doc_heading_count`等の量的指標を
  「指示文書がどれだけ強く禁止・委譲しているか(統制の強さ)」の代理
  指標として使わないこと。実データで反例が確認されている
  (2026-09-14、`agent-trend-playbook`の検証: nuxt/nuxtは948字の文書で
  「AIによる自律コントリビューション・公開文章の作成を禁止」という
  強い禁止型の統制だったのに対し、AutoGPTは3,822字と文字数は4倍だが
  内容は通常のコードスタイル規約で、委譲型の言及はなかった)。文書量は
  整備の手間を示すシグナルであり、統制の強さとは別軸。後者を評価する
  には実際に文書を読む必要がある(agent-trend-radar Issue #27参照)。
- `agent_doc_count`はファイル数ではなく**ディレクトリ数**で数える。
  CLAUDE.md・AGENTS.md・`.cursorrules`をルートに揃えているだけの
  リポジトリ(例: colinhacks/zod)をファイル数で数えると誤って2以上
  (モノレポ扱い)になってしまうため。`agent_doc_count > 1`は「指示文書が
  複数箇所に分散している」ことの決定的シグナルで、モノレポ構成である
  可能性の代理指標として使えるが、`agent_doc_count == 1`だからといって
  モノレポでないとは限らない(指示文書自体を置いていないだけの可能性が
  ある)。より正確なモノレポ判定(package.json workspaces等のワーク
  スペース設定検知)は見送っている(agent-trend-radar Issue #29参照)。

## `manifest.json`

| フィールド | 型 | 説明 |
| --- | --- | --- |
| `dates` | array<string (YYYY-MM-DD)> | `snapshots/` 配下に存在するスナップショット日付の一覧 |
