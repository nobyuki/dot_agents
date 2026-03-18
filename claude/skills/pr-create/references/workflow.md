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

# 未コミットの変更
git status
```

- ベースブランチはデフォルトで `develop`（CLAUDE.md の Main branch 設定に従う）
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

**Test plan の書き方:**
- typecheck / lint / 全テスト PASS は 1 項目にまとめる（個別に分けない）
- テストファイル数・テストケース数などの件数表記は入れない

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

**登録前チェック:** upstream で push 済みかを確認する。upstream が未設定の場合は `git fetch` で origin に同名ブランチがあるかを補助的に確認する。push されていない場合のみ、ユーザーに push を依頼して待つ。

```bash
# upstream が設定済みか確認
BRANCH=$(git branch --show-current)
if git rev-parse --abbrev-ref --symbolic-full-name '@{u}' &>/dev/null; then
  # upstream あり → HEAD がリモートに含まれているか確認
  if [ "$(git rev-list '@{u}'..HEAD --count)" -eq 0 ]; then
    echo "push 済み"
  else
    echo "未 push のローカルコミットあり — ユーザーに push を依頼"
  fi
else
  # upstream 未設定 → origin に同名ブランチがあるか補助確認
  git fetch origin
  if git show-ref --verify --quiet "refs/remotes/origin/$BRANCH"; then
    # 同名ブランチあり → HEAD がリモートに含まれているか確認
    if [ "$(git rev-list "origin/$BRANCH"..HEAD --count)" -eq 0 ]; then
      echo "push 済み"
    else
      echo "未 push のローカルコミットあり — ユーザーに push を依頼"
    fi
  else
    echo "push されていません — ユーザーに push を依頼"
  fi
fi
```

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
