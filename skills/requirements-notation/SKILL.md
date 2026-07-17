---
name: requirements-notation
description: Use when writing or formalizing requirements, user stories, or acceptance criteria — when deciding how to phrase a requirement, which notation (user story / EARS / Given-When-Then) fits, or when requirements are being written as free-form prose without structure.
---

# requirements-notation

要件・受け入れ基準を書くときに適切な記法を選び、references の記述ルールに従って形式化する。/reqd:new の受け入れ基準形式化はこの選択フローに従う。

## 記法選択フロー

| 書こうとしているもの | 記法 | 参照 |
|---|---|---|
| 機能の単位・誰のための何か (分割・優先順位付けの単位) | ユーザーストーリー | `references/user-story.md` |
| システムの応答・挙動の要件 (トリガ・状態・エラー応答) | EARS | `references/ears.md` |
| 受け入れテストとして自動化しうる合否基準 | Given-When-Then | `references/given-when-then.md` |

併用ルール: ユーザーストーリーを WHAT の単位とし、各ストーリーの受け入れ基準を EARS か Given-When-Then で書く (システム挙動の制約 → EARS、シナリオの合否 → GWT)。1つの基準の中で両記法を混ぜない。

## 鉄則

- 自由散文の要件をそのまま受け入れ基準にしない。必ずいずれかの記法に落とす
- 記法に落とせない = 情報が足りない。値や定義を発明せず、未解決事項 (open_questions) に送る

## 掲載ルール (references への記法追加の不変条件)

references/ への記法追加は「科学的根拠」セクションに検証済みの出典があるものに限る。出典が確認できない記法・数値は掲載禁止。
