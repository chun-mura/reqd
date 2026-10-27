---
name: lint
description: 要件定義書・GitHub Issue・貼り付けテキストの曖昧さを検査し、severity 付き指摘と修正案を返す
argument-hint: [ファイルパス / Issue番号・URL / 貼り付けテキスト]
disable-model-invocation: true
---

**パスと引数の解決**: この文書の相対パスは、この SKILL.md があるディレクトリを起点にする。`$ARGUMENTS` が文字のまま残っている場合（Codex・Cursor など）は、この skill を呼び出したメッセージのうち skill 名以外の部分を `$ARGUMENTS` とみなす。

**skill フォールバック**: ambiguity-lint skill が自動発動しない場合は `../ambiguity-lint/SKILL.md` を読んでから進める。

**入力安全性**: `../ambiguity-lint/input-safety.md` を読み、取得・検証・データ境界の全規則に従う。

**入力解決**: 基本的に単一入力として `$ARGUMENTS` を input-safety の規則で解釈する。空白を含むパスは `"file:docs/project memo.md"` のように引用する。
1. 空 -> 何も検査せず、入力候補 (ファイルパス / Issue 番号・URL / 貼り付けテキスト) を案内して終了する
2. GitHub Issue URL -> input-safety の Issue URL 手順で取得する
3. Issue 番号 (`#123` 形式) -> input-safety の Issue 番号手順で取得する
4. `file:<path>` -> `file:` を除いたパスをファイルとして読み込む。存在しない、または読み取り不可の場合は、その旨を指摘して正しいパスの再指定かテキスト貼り付けを促す
5. 存在するファイルパス -> ファイルとして読み込む。読み取り不可の場合は、その旨を指摘して正しいパスの再指定かテキスト貼り付けを促す
6. パスらしいが存在しない入力 -> 自由文にせずパスエラーとする。「パスらしい」の判定は input-safety の「パス判定」節に従う
7. 上記に該当しない非空入力 -> `$ARGUMENTS` 全体を1つの貼り付け・自由記述テキストとして直接処理し、空白で分割しない

GitHub Issue の取得で gh 未インストール・未認証・Issue 不存在となった場合は、その旨を指摘し、本文の貼り付けを促す。

**検査**: ambiguity-lint の観点カタログ (観点1〜8) をすべて適用する。検出ごとに以下を記録する:
- severity (`../ambiguity-lint/SKILL.md` の high / mid / low 定義をそのまま使う)
- 該当箇所の引用
- 観点# (どの観点で検出したか)
- 修正案: 意味を変えない書き換えは具体的な修正文を提案する。入力にない情報の補完が必要な場合は値を発明せず「決めるべき事項」として質問形式で出す

**出力**: severity 降順 (high -> mid -> low) の指摘リストを表示する。**対象ファイルは変更しない (報告のみ)。**

対象が Issue の場合は追加で次の正規形を出力する。実装が分岐する不明点 (high 相当) のみを質問化し、trivial な選択は質問にしない。

```markdown
## needs-info
- [ ] <質問> (source: "<該当箇所の引用>"; 観点#<番号>)

## sensible defaults
- <trivial な選択>: <提案する既定値と理由>
```

**次のステップ案内** (非強制): 指摘を反映した後、要件定義書化するなら `/reqd:new <対象>`。既に要件定義書なら修正後に設計 (`/aidd:design-doc`) へ。
