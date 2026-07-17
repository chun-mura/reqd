---
date: 2026-07-17
---

# reqd — 要件定義プラグイン 設計書

## 目的

要件の元ネタ (メモ、GitHub Issue、brainstorming の対話結果) を構造化された要件定義書に変換し、曖昧さを lint する Claude Code プラグイン。エコシステム内の位置づけは「最上流」: aidd:design-doc・estimate:new・aidd-autopilot triage はすべて要件品質に律速されるため、reqd の出力がそれら下流の入力品質を規定する。

## スコープ (v1)

1. **要件定義書の生成** (`/reqd:new`) — ユーザーストーリー分解 + 受け入れ基準の形式化 (EARS / Given-When-Then)
2. **曖昧さ lint** (`/reqd:lint`) — 要件定義書ファイル・GitHub Issue・貼り付けテキストを対象
3. **記法リファレンス** (skill) — EARS / GWT / ユーザーストーリーの選択ガイドと出典付きリファレンス

### スコープ外 (v1)

- 対話的ヒアリングフロー (superpowers:brainstorming に委譲。棲み分け原則: process = superpowers、成果物 = reqd)
- 要件と実装の乖離検知 (design-sync 相当) — 2回困ってから
- 判断資産の蓄積 skill (uidd の「初期は空」原則に従い、実案件の判断が2回貯まってから)
- hooks / agents — v1 では不要 (stdd と同じ構成レベル)

## 棲み分け

| 隣接 | 境界 |
|---|---|
| superpowers:brainstorming | 発散・対話プロセスは brainstorming。その成果物の形式化・lint が reqd |
| aidd:design-doc | reqd = WHAT (要件)、design-doc = HOW (設計)。reqd の出力パスが design-doc の引数になる |
| estimate (spec-analyzer) | spec-analyzer は見積り用 change_units 抽出。reqd は人間可読の要件定義書を作る。「曖昧性の黙認的解決禁止」の規律と open_questions の概念を共有 |
| stdd | 受け入れ基準 (GWT) が stdd のテストケース導出の入力になる |
| aidd-autopilot triage | triage の needs-info 判定と /reqd:lint の Issue lint は同じ関心事。Issue lint の出力は questions リスト形式 (triage の questions と同形)。autopilot 側の変更はスコープ外 |

## ディレクトリ構造

```
reqd/
├── .claude-plugin/
│   ├── plugin.json          # name: reqd, version: 0.1.0, MIT
│   └── marketplace.json     # chun-mura/reqd 単独マーケットプレイス
├── commands/
│   ├── new.md               # /reqd:new
│   └── lint.md              # /reqd:lint
├── skills/
│   ├── requirements-notation/
│   │   ├── SKILL.md         # 記法選択ガイド + 掲載ルール (出典必須)
│   │   └── references/
│   │       ├── ears.md              # 出典: Mavin et al., RE'09
│   │       ├── given-when-then.md   # 出典: North (BDD 2006), Gherkin 仕様
│   │       └── user-story.md        # 出典: Wake (INVEST 2003), Cohn (2004)
│   └── ambiguity-lint/
│       └── SKILL.md         # lint 基準 (new の自己チェックと lint コマンドが共用)
├── .github/workflows/
│   └── validate.yml         # stdd/uidd と同一パターンの CI
├── CHANGELOG.md
├── LICENSE (MIT)
└── README.md                # 章立て: 導入 → インデックス → 併用推奨 → 運用ルール
```

コマンド (ユーザー起動のワークフロー) と skill (知識・基準、自動参照される) の分離は aidd/stdd の規約に従う。lint 基準を skill に分離するのは、`/reqd:new` の自己チェックと `/reqd:lint` の2箇所から参照されるため (シングルソース)。

## コンポーネント設計

### /reqd:new

- **入力** (`$ARGUMENTS`): ファイルパス / Issue 番号または URL / 自由記述テキスト (混在可)。skill が自動発動しない場合は `skills/requirements-notation/SKILL.md` と `skills/ambiguity-lint/SKILL.md` を直接読んでから進める (stdd:test-design と同じフォールバック規定)
- **処理**:
  1. 入力からユーザーストーリーに分解 (INVEST 基準)
  2. 各ストーリーに受け入れ基準を付与。記法は requirements-notation の選択ガイドに従う (システム挙動 → EARS、受け入れテスト化する基準 → GWT)
  3. スコープ外・未解決事項 (open_questions) を明示。**入力から導出できない項目は勝手に埋めず open_questions に列挙する** (estimate spec-analyzer と同じ規律)
  4. 生成後、ambiguity-lint 基準で自己チェック。振り分けルール: **意味を変えない書き換えで直せるもの (曖昧語彙の言い換え、受動態→能動態、文脈から一意に特定できる主語補完) は自己修正。数値・用語定義など入力にない情報の補完が必要なものは常に open_questions へ** (step 3 の原則の適用)
- **出力**: `docs/requirements/YYYY-MM-DD-<slug>.md`、frontmatter は `date` のみ (status 遷移の主体が v1 に存在しないため status フィールドは持たない)。ディレクトリがなければ作成する
- **出力パスのライフサイクル (サイレント上書きの禁止)**:
  1. 書き込み前に対象パスの存在を確認する
  2. 存在する場合はサイレント上書きせず、既存内容との差分を提示して上書き可否をユーザーに確認する
  3. 拒否された場合は `-2` 等のサフィックス付き別名で保存する (手編集の消失防止)
  4. v1 の改訂ポリシー: 新規生成が既定。既存要件定義書の改訂はユーザーの手動編集で行い、置き換えを明示したい場合のみ旧ファイルの frontmatter に `superseded-by` を手動付与する。上記の存在チェックはこのポリシーの安全弁
- **末尾に次ステップのポインタ**: `/aidd:design-doc <path>`、`/estimate:new <path>` (検出は行わず常時表示する非強制の案内文。未インストールでも壊れない)

### 要件定義書テンプレート (new.md 内に規定)

frontmatter (date) / 背景・目的 / ユーザーストーリー (As a / I want / So that) / 受け入れ基準 (EARS or GWT) / スコープ外 / 未解決事項 (open_questions) / 用語定義

### /reqd:lint

- **入力** (`$ARGUMENTS`): ファイルパス / Issue 番号・URL (gh CLI で取得) / 貼り付けテキスト。skill が自動発動しない場合は `skills/ambiguity-lint/SKILL.md` を直接読んでから進める
- **基準** (ambiguity-lint skill を参照):
  - 曖昧語彙 (「など」「適切に」「高速に」「柔軟に」等の weak words)
  - 測定不能・検証不能な受け入れ基準
  - 主語 (アクター) の欠落、受動態による責任主体の不明化
  - 未定義用語、TBD/TODO の残存
  - 要件間の矛盾・重複
- **出力**: severity (high/mid/low) 付き指摘リスト + 各指摘への修正案。Issue が対象のときは「実装が分岐する不明点」を質問リスト形式でも出力 (autopilot triage の questions と同形)

### ambiguity-lint skill

- lint 観点のカタログ。各観点に出典を付す (掲載ルール: 出典必須、stdd と同一の不変条件)
- 主要出典候補: Berry et al. "From Contract Drafting to Software Specification: Linguistic Sources of Ambiguity" (Ambiguity Handbook)、ISO/IEC/IEEE 29148 (要件の品質特性: unambiguous / verifiable / singular 等)
- **出典は実装時に WebSearch で実在・内容を検証してから掲載する** (stdd の規律)

### requirements-notation skill

- SKILL.md: 「どの記法をいつ選ぶか」の判断のみ。詳細は references/ へ
- 掲載ルール (不変条件): 科学的根拠・原典が確認できない記法・数値は掲載禁止

## エラーハンドリング / フォールバック

- Issue 取得: `gh` 未インストール・未認証・番号不存在の場合は指摘して中断せず、テキスト貼り付けを促す
- ファイルパス入力: 存在しない・読み取り不可の場合は空扱いで続行せず、その旨を指摘して正しいパスの再指定またはテキスト貼り付けを促す (Issue 取得失敗と対称)。複数入力の一部だけ読めない場合は estimate spec-analyzer と同様、読めなかった事実を open_questions に記録して残りで続行する
- `$ARGUMENTS` が空: 何も生成せず、入力候補 (メモのパス / Issue 番号 / 貼り付け) を案内する
- 下流プラグインのポインタ: hook や検出は行わず、コマンド出力末尾に常時案内文を表示するだけ (非ブロック)
- 単体で完結: 他プラグインへの依存は plugin.json に持たない (stdd と同じ疎結合)

## テスト / 検証

- プラグイン自体は Markdown 資産のためビルド・単体テストなし。検証はスモーク:
  1. 実サンプル (このプラグイン自身の要件など) で `/reqd:new` → 出力が テンプレート構造・パス規約に従うこと
  2. 曖昧な文を含む Issue 風テキストで `/reqd:lint` → 既知の曖昧語を検出すること
  3. 出力ファイルを `/aidd:design-doc` と `/estimate:new` に渡して受理されること (連携スモーク)
- CI (`.github/workflows/validate.yml`): stdd/uidd と同一パターンを流用 — JSON 構文検証 (jq)、commands/skills 変更時の version bump 必須チェック、README/CHANGELOG 更新必須チェック
- ローカルでは `claude plugin validate` が使えればそれを、なければ jq でのパース確認を行う

## 運用ルール

aidd README の運用ルールを継承: 1ファイル1関心事 / 2回ルール (昇格は実案件で2回使ってから) / README インデックス更新 / 重複禁止 (他プラグインと重なる領域は委譲ポインタ) / 受動ドキュメント禁止

## 実装順序

1. リポジトリ初期化 (git init、LICENSE、plugin.json、marketplace.json、.github/workflows/validate.yml)
2. skills/requirements-notation (出典の WebSearch 検証 → references 3本 → SKILL.md)
3. skills/ambiguity-lint (出典検証 → SKILL.md)
4. commands/new.md (テンプレート含む)
5. commands/lint.md
6. README.md + CHANGELOG.md
7. スモーク検証 (上記3本) → 修正 → コミット
