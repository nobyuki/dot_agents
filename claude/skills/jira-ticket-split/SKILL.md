---
name: jira-ticket-split
description: >
  JIRA チケットの URL を入力として、粒度チェックを行い、
  分割子チケットを自動生成し、ユーザー承認後に JIRA へ登録するスキル。
  「チケットを分割して」「このチケット大きすぎるから分割して」と言われたときに使う。
---

# JIRA チケット分割

## 前提

### メモリ

スキル実行前に以下の Serena メモリを読み込むこと:

| メモリ名 | 用途 |
|---------|------|
| `project/jira_project_key` | JIRA のプロジェクトキー |

### Atlassian MCP

以下の MCP 操作が利用可能であること。`updateJiraIssue` が利用不可の場合、Phase 4 は手動操作のガイド出力に切り替える。

| 操作 | 用途 | フェーズ |
|------|------|---------|
| `getAccessibleAtlassianResources` | cloudId の取得 | Phase 1 |
| `searchJiraIssuesUsingJql` | 元チケットの詳細取得 | Phase 1 |
| `getJiraProjectIssueTypesMetadata` | issue type の取得 | Phase 3 |
| `createJiraIssue` | 子チケットの作成 | Phase 3 |
| `updateJiraIssue` | description 追記・ラベル付与 | Phase 4 |

## 鉄則

1. **入力は JIRA チケットの URL** — URL から issue key を抽出し、JIRA API で最新情報を取得する。複数 URL を受け付ける
2. **URL バリデーションを最初に実行** — 1件でも無効な URL があれば全体を停止する。issue key を抽出できない URL、重複 URL、`project/jira_project_key` と異なるプロジェクトの URL はすべて無効とする
3. **チケットごとに独立完結** — 複数 URL を受けた場合、Phase 1-4 をチケット単位で完結させる。あるチケットの失敗や分割不要判定が他のチケットの処理をブロックしない
4. **粒度チェックを必ず実行** — task-granularity-check の4項目チェックを適用し、分割根拠をユーザーに提示する
5. **登録はユーザーの明示的な承認を待つ** — 段階的に提示し、承認前に API を呼ばない
6. **元チケットのラベル付与は全子チケット登録後** — 部分登録でラベルを付与しない
7. **jira-ticket のテンプレートに準拠** — 背景・スコープ・完了条件は必須。実装方針・検証方法は必要な場合のみ追加

## ワークフロー

1. **Phase 0: URL バリデーション** — issue key 抽出、重複排除、プロジェクトキー照合
2. **Phase 1: チケット取得・粒度チェック** — JIRA API で詳細取得、4項目チェック適用
3. **Phase 2: 分割案の生成・承認** — 子チケットの description 生成、ユーザーに提示
4. **Phase 3: JIRA 登録** — 承認された子チケットを登録
5. **Phase 4: 元チケットの更新・ラベル付与** — description 追記 + 「AI分割済み」ラベル付与

各フェーズの詳細手順は `references/workflow.md` を参照。

## リファレンス

| ファイル | 内容 | 参照タイミング |
|---------|------|--------------|
| `references/workflow.md` | ワークフロー詳細手順（URL バリデーション・粒度チェック・分割案生成・登録・ラベル付与） | Phase 0-4: 各フェーズの実行時 |
| `references/split-template.md` | 分割後チケットの description 生成ルール（セクション選択指針・分割固有ルール） | Phase 2: 分割案生成時 |

## 関連スキル

### 参照先

| スキル名 | 参照タイミング | 用途 |
|---------|--------------|------|
| `jira-ticket` | Phase 2: description テンプレート・落とし穴チェック | テンプレート準拠・品質チェック |
| `task-granularity-check` | Phase 1: 粒度チェック、Phase 2: 分割後チケットの粒度再検証 | 4項目チェック・分割評価基準・スコープトリアージ |
