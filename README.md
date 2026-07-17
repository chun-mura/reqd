# reqd

要件の元ネタ (メモ・GitHub Issue・対話の結果) を構造化された要件定義書に変換し、曖昧さを lint する Claude Code プラグイン。

**棲み分け**: process (発散・対話ヒアリング) は superpowers (brainstorming)、設計 = HOW は aidd (design-doc)、見積りは estimate、テスト設計は stdd。要件 = WHAT の形式化と品質検査が reqd。

## 導入

```bash
/plugin marketplace add chun-mura/reqd
/plugin install reqd@reqd

# 併用推奨: superpowers (brainstorming) + aidd (design-doc) + estimate (見積り)
```

## インデックス

### Commands (タスク特化プロンプト)

| Command | 用途 |
|---|---|
| `/reqd:new` | 検証済み識別子から Issue の title/body/comments またはファイルを取得し、衝突しないデータ境界内で要件定義書を生成する |
| `/reqd:lint` | 検証済み識別子から Issue の title/body/comments またはファイルを取得し、衝突しないデータ境界内で曖昧さ検査と修正案を返す |

### Skills (自動トリガする知見)

| Skill | いつ発動するか |
|---|---|
| `requirements-notation` | 要件・受け入れ基準を書く前。ユーザーストーリー / EARS / Given-When-Then の選択フローと表示可能なテンプレートで自由散文を形式化する |
| `ambiguity-lint` | 要件・Issue・受け入れ基準を書くとき/レビューするとき。曖昧語彙・検証不能な基準・アクター欠落・未定義用語・矛盾を観点カタログで検出する |

## 併用推奨

- **superpowers**: 発散・対話ヒアリング (brainstorming) はそちらに委譲する。その成果物を `/reqd:new` への入力にする
- **aidd**: `/reqd:new` の出力パスを `/aidd:design-doc` の引数に渡す (WHAT → HOW)
- **estimate**: `/reqd:new` の出力パスを `/estimate:new` に渡す (要件 → 見積り)
- **stdd**: 受け入れ基準 (Given-When-Then) を stdd のテストケース導出への入力に使う

## 運用ルール

1. 1ファイル1関心事 (冒頭に「いつ使うか」を書く)
2. 2回ルール (2回以上使ったプロンプトのみ資産に昇格)
3. 資産変更時は README インデックス + CHANGELOG + plugin.json version を同時更新
4. 重複禁止 (superpowers/aidd/estimate/stdd と重複する機能は作らずポインタのみ)
5. 受動ドキュメント禁止 (知見は skills=自動トリガ か hooks=強制 に置く)

> ⚠️ このプラグインは実プロジェクトでの検証が未実施である。
