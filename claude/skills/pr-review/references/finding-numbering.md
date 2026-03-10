# Finding ナンバリング規則

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

人間レビュアーおよび Claude Code（ローカル PC）は、PR に投稿する際のアカウントである GitHub `user.login` をそのまま使用する。Claude Code がローカルで作成したレビューは人間レビュアー名義で PR に投稿されるため、同一の識別子を使う。ボット系（Copilot, custom-claude-bot, Codex）は固定 Prefix を使用する。

## ラウンドの定義

ラウンドは `gh api repos/{owner}/{repo}/pulls/{number}/reviews` で取得できる **review submit event** 単位でカウントする。1回の submit（state: `COMMENTED` / `APPROVED` / `CHANGES_REQUESTED`）を1ラウンドとする。

**ラウンド番号の採番**: `submitted_at` 昇順（古い順）で R1, R2, ... と採番する。レビュアーごとに独立してカウントする。

**issue comment の扱い**: issue comment（PR 本文下のコメント）はラウンドとしてカウントしない。ただし、issue comment の投稿時刻が review submit event の `submitted_at` から前後5分以内の場合、同一ラウンドの補足として扱う。複数の review submit event が該当する場合は、最も `submitted_at` が近いものに紐付ける。同距離の場合は `review.id` 昇順で先のものに紐付ける。どの review event にも該当しない単独の issue comment はラウンドに含めない。

## 通し番号

ラウンド内での Finding の連番（1始まり）。

## 統合ルール

他レビュアーの指摘を自身の Finding に統合した場合:

- **統合先**: 通常通り Finding として記載
- **統合元**: 元 ID を閉鎖せず、Status と統合先の実 ID を必須記載する（テンプレート変数ではなく、必ず `user1-R4.1` のような実 ID を記入すること）

```markdown
## PR 上のレビューコメント確認

| ID | レビュアー | 指摘 | Status | 検証結果 |
|----|-----------|------|--------|---------|
| CP-R4.1 | Copilot | 同一ユーザーの複数組織ケース | merged | Merged into user1-R4.1 |
| CX-R1.1 | Codex | 同一ユーザーの複数組織ケース | merged | Merged into user1-R4.1 |
| CP-R4.2 | Copilot | ドキュメントの定数更新 | open | 事実 — Finding user1-R4.2 として記載 |
| CP-R3.1 | Copilot | エンティティ解決が不完全 | resolved | 対応済 — 該当箇所修正済み |
| CP-R3.2 | Copilot | 制限付きロールでサインイン可能 | invalid | 今回は対象外 — PROJ-456 で別途対応予定 |
```

## Status 定義と遷移ルール

| Status | 意味 |
|--------|------|
| `open` | 未対応（自身の Finding として管理中） |
| `resolved` | コード修正により対応済み |
| `merged` | 他の Finding に統合済み |
| `invalid` | 事実と異なる、対象外、または該当しない |

**遷移ルール**: `open` → `resolved` / `merged` / `invalid` の一方向のみ。一度 `resolved` / `merged` / `invalid` になった Finding は再オープンしない。修正が不十分だった場合や回帰が発見された場合は、新しいラウンドで新規 Finding として起票する。

```
open ──→ resolved
     ├─→ merged
     └─→ invalid

（逆方向の遷移なし。再発時は新規 Finding を起票）
```

## 本文中の参照

```markdown
CP-R4.1 および CX-R1.1 と同一の指摘であり、user1-R4.1 に統合した。
```
