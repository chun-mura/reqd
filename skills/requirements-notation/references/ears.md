# EARS (Easy Approach to Requirements Syntax)

いつ使うか: システムの応答・挙動要件 (トリガ・状態・オプション機能・望ましくない事象への応答) を1文で厳密に書くとき。

## 5つのパターン

| パターン | テンプレート | 使いどころ |
|---|---|---|
| Ubiquitous (常時) | `The <system> shall <response>` | 常に成り立つ性質 |
| Event-driven | `When <trigger>, the <system> shall <response>` | イベント起点の応答 |
| State-driven | `While <state>, the <system> shall <response>` | 特定状態中の挙動 |
| Optional feature | `Where <feature is included>, the <system> shall <response>` | 構成依存の機能 |
| Unwanted behaviour | `If <unwanted condition>, then the <system> shall <response>` | エラー・異常系への応答 |

複合パターン: `While <state>, when <trigger>, the <system> shall <response>` のように組み合わせてよい。日本語では「<状態> において <トリガ> のとき、<システム> は <応答> しなければならない」の形で書き、状態/トリガ/応答の要素を落とさない。

## 導出手順

1. 要件文からトリガ (イベント)・状態・望ましくない条件を特定する
2. 該当パターンを選ぶ。異常系は必ず Unwanted behaviour として正常系と別文に分ける
3. `<system>` と `<response>` を具体名で埋める。応答は観測可能・検証可能な表現にする (「適切に処理する」は不可)
4. 1文1要件 (singular)。「かつ」「また」で複数の応答を繋げない

## 科学的根拠

- Mavin, A., Wilkinson, P., Harwood, A., Novak, M. (2009). "Easy Approach to Requirements Syntax (EARS)". Proc. 17th IEEE International Requirements Engineering Conference (RE'09), pp. 317-322. DOI: 10.1109/RE.2009.9 — 原典。Rolls-Royce のジェットエンジン制御要件への適用で、曖昧さ・重複・過剰な複雑さの減少を報告
- Mavin, A., Wilkinson, P. (2010). "Big Ears (The Return of Easy Approach to Requirements Engineering)". Proc. 18th IEEE International Requirements Engineering Conference (RE'10), pp. 277-282. DOI: 10.1109/RE.2010.39 — 適用の追試報告
