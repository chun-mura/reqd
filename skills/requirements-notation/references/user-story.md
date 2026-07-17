# ユーザーストーリー / INVEST

いつ使うか: 要件を分割・優先順位付けの単位に分解するとき。

## 形式

As a <役割>, I want <機能>, so that <価値>.
(日本語: <役割> として、<価値> のために、<機能> がほしい)

so that (価値) が書けないストーリーは要件の根拠が不明。値を発明せず未解決事項 (open_questions) に送る。

## INVEST チェックリスト

| 基準 | 問い |
|---|---|
| Independent | 他ストーリーと順序依存なく実装できるか |
| Negotiable | 実装詳細を固定しすぎていないか (WHAT であって HOW でない) |
| Valuable | ユーザー・顧客への価値が so that に書けているか |
| Estimable | 見積れる程度に具体か (/estimate:new に渡せるか) |
| Small | 1イテレーションに収まるか |
| Testable | 受け入れ基準 (EARS / Given-When-Then) が書けるか |

## 分割の目安

Small / Independent を満たさないストーリーは、次のいずれかの軸で分割する: ワークフローの段階 / 操作対象のバリエーション / 正常系と異常系。

## 科学的根拠

- Wake, B. (2003). "INVEST in Good Stories, and SMART Tasks". 2003-08-17. https://xp123.com/invest-in-good-stories-and-smart-tasks/ — INVEST 基準の原典
- Cohn, M. (2004). "User Stories Applied: For Agile Software Development". Addison-Wesley. — ユーザーストーリー形式と受け入れ基準の体系化
