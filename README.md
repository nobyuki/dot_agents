# dot_agents

AI コーディングエージェントの設定・カスタマイズ集。

## 構成

| ディレクトリ | 内容 |
|------------|------|
| `claude/` | Claude Code のカスタムスキル |
| `codex/` | Codex の設定（準備中） |

## Claude Code

### スキル

`claude/skills/` 配下にカスタムスキルを格納しています。`~/.claude/skills/` にコピーまたはシンボリックリンクして使用します。

### Serena メモリへの依存

一部のスキルは [Serena](https://github.com/SIRIUS-CADO/serena) MCP サーバーのプロジェクトメモリに、プロジェクト固有の情報が登録されていることを前提としています。スキル自体は汎用的に書かれており、プロジェクト固有のパス・設定・命名規則はメモリから取得します。

#### 必要なメモリ一覧

| メモリ名 | 用途 | 参照するスキル |
|---------|------|--------------|
| `project/plan_docs_location` | 計画書の保存先ディレクトリパスとコミットルール | plan-feature, plan-doc-update, plan-doc-review, impl-driver, verify-review, pr-review |
| `project/doc_survey_targets` | ドキュメント更新時のサーベイ対象ファイルリスト | plan-doc-update |
| `project/architecture_knowledge` | アーキテクチャ構成、レイヤー配置、開発コマンド、ドキュメントディレクトリパス、コード変更パターン、既存計画書の参照 | plan-feature, plan-doc-update, impl-driver |
| `project/jira_project_key` | JIRA のプロジェクトキー | jira-ticket |

#### メモリの登録方法

Serena の `write_memory` ツールを使って各メモリを登録します。内容はプロジェクトに合わせて記述してください。

メモリが未登録の場合、該当スキルはメモリ参照時にエラーまたは空の応答になります。スキルを利用する前に、対象プロジェクトで必要なメモリを登録してください。
