---
name: task-granularity-check
description: >
  タスクやPRの粒度が適切かを判定し、分割や後続タスク切り出しを提案する内部リファレンススキル。
  ユーザーが直接呼び出すのではなく、計画作成・チケット起票・PRレビュー等のスキルから
  タスク分割判断が必要なタイミングで自動参照される。
  対象スキル: plan-feature, plan-doc-update, plan-doc-review, jira-ticket, jira-ticket-split, jira-sprint-report, pr-review, verify-review
---

# タスク粒度チェック

## 目的

計画作成、チケット起票、計画レビュー、PRレビューの各フェーズにおいて、コンテキストの肥大化と要件解釈のズレを防ぐための粒度統制とスコープ管理ルール。

## 適用コンテキスト

| コンテキスト | 適用タイミング | 参照元スキル |
|------------|--------------|-------------|
| PLANNING_AND_TICKETING | 計画作成・チケット起票時のタスク分割基準 | plan-feature, plan-doc-update, jira-ticket, jira-ticket-split, jira-sprint-report |
| PRE_IMPLEMENTATION | コード生成前の品質ゲート | plan-feature, plan-doc-review |
| PR_REVIEW | PR の粒度・混入物評価 | pr-review, verify-review |

## 判定フロー（概要）

1. **要件構造の確認** — Purpose が単一目的か、受け入れ条件が整理されているか
2. **分割評価** — 4つのチェック（単一目的・単一振る舞い・安全なRevert・単一レビューコンテキスト）
3. **スコープトリアージ** — メインタスクに含めるか、後続タスクに切り出すかを判定
4. **アクション** — 分割提案、後続チケット提案、または同一単位維持（理由付き）

## リファレンス

| ファイル | 内容 | 参照タイミング |
|---------|------|--------------|
| `references/granularity-rules.md` | 要件定義構造、分割評価基準、スコープトリアージロジック、レビュー優先度、フォールバックルール | 粒度判断が必要な全タイミング |

## 関連スキル

### 参照元

以下のスキルが粒度判断時にこのスキルを参照する:

| スキル名 | 参照タイミング |
|---------|--------------|
| `plan-feature` | 実装ステップの分割粒度を決定する際 |
| `plan-doc-update` | Step単位の分割粒度を決定する際 |
| `plan-doc-review` | 計画書の実装ステップの粒度を評価する際 |
| `jira-ticket` | 要件整理時にチケットの分割要否を判断する際 |
| `pr-review` | PRの粒度・混入物を評価する際 |
| `verify-review` | レビュー指摘の中でタスク切り出し提案の妥当性を検証する際 |
| `jira-sprint-report` | スプリントレポート生成時に各チケットの粒度を判定する際 |
| `jira-ticket-split` | Phase 1: 元チケットの粒度チェック、Phase 2: 分割後チケットの粒度再検証 |
