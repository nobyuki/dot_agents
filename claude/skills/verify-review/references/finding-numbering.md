# Finding ナンバリング規則（検証コンテキスト）

> pr-review の Finding ナンバリング規則をベースに、レビュー検証コンテキスト向けに調整したもの。
> 原本: `~/.claude/skills/pr-review/references/finding-numbering.md`

## フォーマット

```
{レビュアー識別子}-R{ラウンド}.{通し番号}
```

## レビュアー識別子

| 種別 | 識別子 | 例 |
|------|--------|-----|
| 人間レビュアー / Claude Code（ローカル） | GitHub ユーザー名（`user.login` を正とする） | `user1-R4.1` |
| Copilot | `CP` | `CP-R1.3` |
| custom-claude-bot | `CB` | `CB-R1.2` |
| Codex | `CX` | `CX-R1.1` |

人間レビュアーおよび Claude Code（ローカル PC）は、PR に投稿する際のアカウントである GitHub `user.login` をそのまま使用する。ボット系（Copilot, custom-claude-bot, Codex）は固定 Prefix を使用する。

## ラウンドの定義

ラウンドは `gh api repos/{owner}/{repo}/pulls/{number}/reviews` で取得できる **review submit event** 単位でカウントする。1回の submit（state: `COMMENTED` / `APPROVED` / `CHANGES_REQUESTED`）を1ラウンドとする。

**ラウンド番号の採番**: `submitted_at` 昇順（古い順）で R1, R2, ... と採番する。レビュアーごとに独立してカウントする。

**issue comment の扱い**: issue comment（PR 本文下のコメント）はラウンドとしてカウントしない。ただし、issue comment の投稿時刻が review submit event の `submitted_at` から前後5分以内の場合、同一ラウンドの補足として扱う。複数の review submit event が該当する場合は、最も `submitted_at` が近いものに紐付ける。同距離の場合は `review.id` 昇順で先のものに紐付ける。どの review event にも該当しない単独の issue comment はラウンドに含めない。

## 通し番号

ラウンド内での主張（claim）の連番（1始まり）。検証テーブルの各行に対応する。

## 検証テーブルでの使用

verify-review では、検証対象のレビューコメントに含まれる各主張に対して、元のレビュアーとラウンドに基づく ID を付与する。

```markdown
### 各主張の検証

| # | 主張 | 判定 | 根拠 | スコープ判定 |
|---|------|------|------|-------------|
| CP-R1.1 | エラーハンドリングが不足 | ✅ 正確 | src/handler.ts:42 に try-catch なし | スコープ内 |
| CP-R1.2 | 型定義が不整合 | ❌ 不正確 | types.ts:15 で正しく定義済み | スコープ内 |
```

## 複数レビュアーの検証

同一 PR に対する複数レビュアーのコメントを検証する場合、各レビュアーの ID 体系をそのまま使用する。総合評価では ID で相互参照できる。

```markdown
| # | 主張 | 判定 | 根拠 | スコープ判定 |
|---|------|------|------|-------------|
| CP-R1.1 | 同一ユーザーの複数組織ケース | ✅ 正確 | ... | スコープ内 |
| CX-R1.1 | 同一ユーザーの複数組織ケース | ✅ 正確 | CP-R1.1 と同一指摘 | スコープ内 |
```
