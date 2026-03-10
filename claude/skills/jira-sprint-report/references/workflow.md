# ワークフロー詳細

## Phase 1: スプリント特定・チケット取得

### 1-1. スプリント名の確定

ユーザーからスプリント名を受け取る。指定がない場合は確認する。

### 1-2. JIRA チケット取得

1. `getAccessibleAtlassianResources` で cloudId を取得
2. JQL でチケットを検索:
   ```
   project = {projectKey} AND sprint = "{sprintName}"
   ```
3. 取得フィールド: `summary`, `description`, `status`, `issuetype`, `priority`, `assignee`, `labels`, `subtasks`
   - `status` の返却には `statusCategory`（`To Do` / `In Progress` / `Done`）が含まれる
4. `maxResults: 50` で取得し、`isLast: false` の場合は `nextPageToken` でページネーション

### 1-3. チケット整理

取得したチケットを JIRA の `statusCategory` に基づいて分類:
- **未完了**: statusCategory が `To Do` または `In Progress` のチケット
- **完了**: statusCategory が `Done` のチケット
- **親ストーリー**: `subtasks` フィールドが空でなく、子タスクが同スプリントに存在するチケット

> **注意**: ステータス名（「作業前」「進行中」等）はプロジェクトごとに異なるため、個別のステータス名ではなく `statusCategory` で判定すること。課題タイプ名も同様に、名前のハードコードではなく `subtasks` の有無で親子関係を判定する。

## Phase 2: 粒度チェック

`task-granularity-check` スキルの `references/granularity-rules.md` に従い、各チケットに以下を適用:

### 2-1. 4項目チェック

各チケットに対して以下を判定:

| # | チェック項目 | 基準 |
|---|------------|------|
| 1 | `is_single_purpose` | 目的が一文で説明でき、ついでに行う変更が含まれていないか |
| 2 | `is_single_behavior` | 守るべきテスト観点が1テーマに収まっているか |
| 3 | `is_safe_to_revert` | この単位単体をRevertした場合、システムとして意味が通るか |
| 4 | `is_single_review_context` | レビュアーが複数ドメインのコンテキストスイッチなしにレビューできるか |

### 2-2. 判定ラベル

チェック結果に基づき以下のラベルを付与:

| ラベル | 条件 |
|-------|------|
| PASS | 全4項目が true |
| WARN | 1項目が要注意だが、不可分の正当な理由がある |
| SPLIT_RECOMMENDED | 1項目以上が false で、分割が自然にできる |
| NEEDS_DETAIL | description が空またはスコープ不明で評価不能 |
| SKIP | 親ストーリーなど粒度判定の対象外 |

### 2-3. 分割要否判定

各チケットに対して分割要否を判定:

| 判定 | 条件 |
|------|------|
| KEEP | 全項目 true、または不可分の理由がある（KEEP_TOGETHER_IF_INSEPARABLE） |
| SPLIT | false 項目があり、分割が自然かつ有益 |
| N/A | 粒度判定対象外 |

分割判定時は `task-granularity-check` の以下を参照:
- デフォルト分割パターン（仕様変更 AND リファクタリング等）
- スコープトリアージロジック（ROUTE_TO_MAIN_TASK / ROUTE_TO_FOLLOW_UP_TASK_CANDIDATE）
- フォールバックルール（REDUCE_SCOPE → SPLIT_TASK → EXTRACT_FOLLOW_UP_TICKET）

## Phase 3: SP概算

`references/sp-estimation-guide.md` の基準に従い、各チケットにSP概算を付与する。

### 3-1. 概算手順

1. チケットの description から影響範囲・複雑度を把握
2. SP基準テーブルと照合し、概算値を決定
3. 根拠を1行で記述（例: 「API+UI+テスト。中規模機能追加」）
4. description に工数見積もりが記載されている場合はそれを参考値として活用

### 3-2. 集計

- 親ストーリーのSPは「—」とし、子タスク合算値を備考に記載
- スコープ未定義（NEEDS_DETAIL）のチケットは「?」とし、合計から除外
- 未完了合計 / 完了合計 / 消化率を算出

## Phase 4: レポート生成

`references/report-template.md` のテンプレートに従い、レポートを `/tmp/` に出力する。

### 4-1. 出力先

- ユーザー指定のパスがあればそれに従う
- なければ `/tmp/sprint-report-{sprint名}.md`

### 4-2. レポート構成

テンプレートの全セクションを出力:
1. ヘッダー（スプリント名、日付、チケット数）
2. サマリー（判定別集計）
3. 分割要否判定サマリー
4. ストーリーポイント概算一覧
5. 未完了チケット詳細分析
6. 完了チケット簡易分析
7. 推奨アクション一覧
8. 判定基準リファレンス
