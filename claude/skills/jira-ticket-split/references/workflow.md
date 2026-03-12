# JIRA チケット分割ワークフロー詳細

## Phase 0: URL バリデーション

1. **ユーザーから JIRA チケットの URL を受け取る**（1件以上）
2. **各 URL を検証** — 以下のいずれかに該当する URL は無効:
   - issue key を抽出できない URL
   - 重複 URL
   - `project/jira_project_key` と異なるプロジェクトキーの URL
3. **1件でも無効な URL があれば全体を停止** — バリデーション結果を表示して終了。ユーザーに修正してから再実行してもらう

```markdown
## URL バリデーション失敗

| URL | issue key | 状態 |
|-----|-----------|------|
| https://xxx.atlassian.net/browse/PROJ-101 | PROJ-101 | ✅ 有効 |
| https://xxx.atlassian.net/browse/OTHER-55 | OTHER-55 | ❌ プロジェクトキー不一致（期待: PROJ） |
| https://invalid-url | — | ❌ issue key を抽出できない |

無効な URL が含まれています。修正してから再実行してください。
```

4. **全 URL が有効な場合のみ続行** — 処理対象を提示して確認。以降、各チケットに対して Phase 1-4 を独立して実行する

## Phase 1: チケット取得・粒度チェック

1. **JIRA API で元チケットの詳細を取得**:
   - `getAccessibleAtlassianResources` で cloudId を取得（初回のみ）
   - JQL `key = PROJ-101` で description, summary, status, issuetype, priority, labels を取得
2. **`task-granularity-check` の4項目チェックを適用**:
   - `is_single_purpose` / `is_single_behavior` / `is_safe_to_revert` / `is_single_review_context`
   - デフォルト分割パターン（仕様変更 AND リファクタリング等）の検知
3. **粒度チェック結果をユーザーに提示**:

```markdown
## 粒度チェック結果

### PROJ-101: ○○機能の実装

| # | チェック項目 | 判定 | 理由 |
|---|-------------|------|------|
| 1 | is_single_purpose | **false** | API実装とUI実装の2目的が混在 |
| 2 | is_single_behavior | **false** | テスト観点がAPIとUIで分離可能 |
| 3 | is_safe_to_revert | true | |
| 4 | is_single_review_context | **false** | バックエンドとフロントエンドのコンテキストスイッチが必要 |

**判定: 分割推奨** — 3項目が false。API / UI / テストに分割可能。

分割案の生成に進みますか？
```

4. **ユーザーが続行を承認したら Phase 2 へ** — 全項目 true で分割不要と判定された場合はスキップし、次のチケットへ進む

## Phase 2: 分割案の生成・承認

1. **子チケットの description を生成** — 元チケットの description 全文と粒度チェック結果を元に:
   - `split-template.md` のセクション選択指針に従う
   - `jira-ticket/references/pitfalls.md` のチェックを適用
   - 生成した子チケットに対して `task-granularity-check` の4項目チェックで粒度を再検証
2. **分割案をユーザーに提示**:

```markdown
### PROJ-101 の分割案

#### 子チケット 1/3: ○○の API 実装
- **issue type**: タスク

<description>
## 背景
（元チケット PROJ-101 の分割。○○機能の API レイヤー実装を担当）

## スコープ
### 本チケットで実施
1. ...

### スコープ外（別チケットで対応）
- UI 実装は PROJ-101 分割の別チケットで対応

## 実装方針
（元チケットの方針から該当部分を引用。省略可）

## 完了条件
1. ...
</description>

#### 子チケット 2/3: ...
（同様）

---
この分割案で進めますか？（修正指示があればお伝えください）
```

3. **修正要求があれば反映して再提示**
4. **ユーザーが承認したら Phase 3 へ** — スキップを選択した場合は次のチケットへ

## Phase 3: JIRA 登録

1. **登録対象の最終確認を提示**:

```markdown
## PROJ-101 の登録確認

| # | 子チケットタイトル | issue type |
|---|-----------------|-----------|
| 1 | ○○の API 実装 | タスク |
| 2 | ○○の UI 実装 | タスク |
| 3 | ○○のテスト追加 | タスク |

3 チケットを登録し、PROJ-101 に「AI分割済み」ラベルを付与します。
「登録して」で実行します。
```

2. **ユーザーの「登録して」を待つ** — 承認前に API を呼ばない
3. **issue type を取得** — `getJiraProjectIssueTypesMetadata` で取得（初回のみ）
4. **子チケットを登録** — 各子チケットを `createJiraIssue` で登録し、チケットキーを記録

## Phase 4: 元チケットの更新・ラベル付与

1. **`updateJiraIssue` を1回呼び出し** — 以下を同時に更新:
   - **description**: 既存 description の末尾に分割先参照を追記
   - **labels**: 既存ラベルを維持したまま `AI分割済み` を追加
   - ステータスは変更しない（クローズ判断はユーザーに委ねる）

追記内容:
```markdown
---
## 分割先チケット
本チケットは以下に分割されました:
- PROJ-201: ○○の API 実装
- PROJ-202: ○○の UI 実装
- PROJ-203: ○○のテスト追加
```

2. **更新失敗時は手動更新のガイドを表示** — 次のチケットの処理に進む
3. **結果サマリーを表示**:

```markdown
## PROJ-101 の分割完了

### 作成されたチケット
| 子チケット | タイトル | URL |
|-----------|--------|-----|
| PROJ-201 | API実装 | ... |
| PROJ-202 | UI実装 | ... |
| PROJ-203 | テスト追加 | ... |

### 元チケット更新
- PROJ-101: ✅ description 追記 + `AI分割済み` ラベル付与
```

全チケットの処理完了後に全体サマリーを表示する（複数 URL 入力時）:

```markdown
## 全体サマリー

| # | チケット | 結果 | 子チケット |
|---|---------|------|-----------|
| 1 | PROJ-101 | ✅ 分割完了（3件） | PROJ-201, PROJ-202, PROJ-203 |
| 2 | PROJ-102 | ⏭️ 分割不要 | — |
| 3 | PROJ-103 | ❌ 登録失敗（2/3件成功） | PROJ-204, PROJ-205 |
```
