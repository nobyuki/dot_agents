# PR 作成ワークフロー詳細

## Phase 1: ブランチ状態の確認

以下を並列で取得する:

```bash
# ベースブランチからの全コミット
git log <base>..HEAD --oneline

# 差分の統計
git diff <base>..HEAD --stat

# 差分の詳細（必要に応じて）
git diff <base>..HEAD

# リモート追跡状態
git rev-parse --abbrev-ref --symbolic-full-name '@{u}'

# 未コミットの変更
git status
```

- ベースブランチはデフォルトで `develop`（CLAUDE.md の Main branch 設定に従う）
- リモートに push されていない場合、ユーザーに push を依頼して待つ
- 未コミットの変更がある場合、ユーザーに確認する

## Phase 2: PR 案の作成

`/tmp/pr-{branch-short-name}.md` に PR 案を出力する。

**出力フォーマット:**

```markdown
## Title

<type>(<scope>): <簡潔な件名（70文字以内）>

## Body

## Summary

- <変更内容を1-3行で箇条書き>

## Changes（差分が多い場合のみ）

| ファイル | 変更内容 |
|---------|---------|

## Notes（補足が必要な場合のみ）

- <レビュアーに伝えるべき判断・背景>

## Test plan

- [x] <検証済み項目>
- [ ] <未検証項目>
```

**セクション選択の指針:**
- Summary は必須
- Changes は差分ファイルが多い場合（目安5ファイル以上）に追加
- Notes はレビュアーが判断に迷いそうな点がある場合に追加
- Test plan は必須

**type の選び方:** impl-driver スキルのコミットメッセージ規約に準拠（feat, fix, refactor, test, docs, chore）

## Phase 3: レビュー反映（繰り返し）

ユーザーやレビューツール（Codex 等）のフィードバックを受けて PR 案を更新する。

1. 指摘の妥当性を差分に照合して検証
2. 妥当な指摘のみ反映して `/tmp` のファイルを更新
3. ユーザーが満足するまで繰り返す

## Phase 4: PR 登録

ユーザーの「作成して」「登録して」を待ってから実行する。

```bash
gh pr create \
  --base develop \
  --title "<Title>" \
  --body "$(cat <<'EOF'
<Body の内容>
EOF
)"
```

- ユーザーが `--draft` を指示した場合は `--draft` フラグを付ける
- 作成後、PR URL を返す
