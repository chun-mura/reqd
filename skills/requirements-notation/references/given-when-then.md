# Given-When-Then

いつ使うか: 受け入れ基準を「実行可能なテストになりうる合否条件」として書くとき。

## 構造

- **Given** <前提状態>
- **When** <単一のアクション・イベント>
- **Then** <観測可能な結果>
- (**And** で各節を補強できる)

## 記述ルール

1. 1シナリオ1振る舞い。When は1つだけ (複数アクションが必要ならシナリオを分割する)
2. Then は外から観測可能な結果のみを書く (内部実装の状態を書かない)
3. 宣言的に書く。UI 操作手順 (「ボタンを押す」) ではなく意図 (「注文を確定する」)
4. Given に検証を書かない。Then に前提を書かない
5. 具体値を使う (「多数の」ではなく「100件の」)

## 導出手順

1. ストーリーの合否を分ける状況を列挙する (正常・境界・異常)
2. 状況ごとに Given/When/Then を1シナリオずつ書く
3. 導出したシナリオはそのまま stdd のテストケース導出・superpowers:test-driven-development の RED への入力になる

## 科学的根拠

- North, D. (2006). "Introducing BDD". Better Software Magazine, March 2006. https://dannorth.net/blog/introducing-bdd/ — Given-When-Then テンプレートの原典
- Gherkin Reference. Cucumber 公式ドキュメント. https://cucumber.io/docs/gherkin/reference — Given/When/Then 構文の仕様
