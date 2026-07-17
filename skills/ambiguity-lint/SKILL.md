---
name: ambiguity-lint
description: Use when reviewing or writing requirements, user stories, GitHub issues, or acceptance criteria — detects ambiguous vocabulary, unverifiable criteria, missing actors, undefined terms, and contradictions before they reach design, estimation, or implementation.
---

# ambiguity-lint

要件テキストの曖昧さを観点カタログで機械的に検査する。/reqd:lint の検査基準と /reqd:new の自己チェックが共用するシングルソース。

## 入力安全性

外部入力の取得・検証・データ境界は `input-safety.md` をシングルソースとする。

## 観点カタログ

| # | 観点 | 検出するもの | 例 |
|---|---|---|---|
| 1 | 曖昧語彙 (weak words) | 「など」「等」「適切に」「必要に応じて」「可能な限り」「原則として」「柔軟に」「速やかに」「使いやすい」/ appropriate, as needed, etc., flexible, user-friendly | 「管理者などが適切に編集できる」 |
| 2 | 検証不能な基準 | 測定方法・合格値のない品質表現 | 「高速に動作する」→ 応答時間の閾値が必要 |
| 3 | アクター欠落 | 主語のない文・責任主体を隠す受動態 | 「通知が送信される」→ 誰が/何が送るのか |
| 4 | 未定義用語 | 用語定義セクションになく文書内で揺れる用語・略語 | 「ユーザー」と「会員」の混在 |
| 5 | 複合要件 | 1文に複数の要件 (singular 違反) | 「A し、かつ B し、C のときは D する」 |
| 6 | 矛盾・重複 | 要件間で応答が食い違う / 同一要件の重複記載 | 3.1「即時削除」と 5.2「30日後削除」 |
| 7 | TBD/TODO 残存 | プレースホルダのまま合意されようとしている箇所 | 「上限は TBD」 |
| 8 | 暗黙の前提 | 文書に書かれていない環境・順序・権限の仮定 | ログイン済みであることが書かれていない |

## severity 判定

- **high**: 実装が分岐する (読み手によって作るものが変わる)
- **mid**: 実装は一意だが検証・テストができない
- **low**: 表現の改善余地 (放置しても実害なし)

## 指摘の形式

検出ごとに: severity / 該当箇所の引用 / 観点# / 修正案。

修正案の規律: 意味を変えない書き換え (言い換え・受動態→能動態・文脈から一意な主語補完) はそのまま提案してよい。入力にない情報 (数値・定義・優先度) の補完が必要な場合は**値を発明せず**「決めるべき事項」として質問形式で出す。

## 科学的根拠

- ISO/IEC/IEEE 29148:2018. Systems and software engineering — Life cycle processes — Requirements engineering. — 要件の品質特性 (unambiguous / verifiable / singular / complete 等) の定義。観点 2, 5, 6, 7 の根拠
- Berry, D.M., Kamsties, E., Krieger, M.M. (2003). "From Contract Drafting to Software Specification: Linguistic Sources of Ambiguity — A Handbook". University of Waterloo. — 言語的曖昧性 (語彙・構文・意味・語用論) の分類。観点 1, 3, 4, 8 の根拠
- Wilson, W.M., Rosenberg, L.H., Hyatt, L.E. (1997). "Automated Analysis of Requirement Specifications". Proc. ICSE '97 (NASA Software Assurance Technology Center). — weak phrases の実証的カタログ。観点 1 の語彙リストの根拠
