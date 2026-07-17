# reqd 要件定義プラグイン Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 要件の元ネタ (メモ・GitHub Issue・自由記述) を構造化された要件定義書に変換し、曖昧さを lint する Claude Code プラグイン reqd (v0.5.0 相当まで) を構築する。

**Architecture:** Markdown 資産のみのプラグイン (実行コードなし)。commands/ (ユーザー起動ワークフロー: new / lint) + skills/ (自動発動する知識: requirements-notation / ambiguity-lint) の2層。lint 基準は skill に一元化し、`/reqd:new` の自己チェックと `/reqd:lint` の両方から参照する (シングルソース)。設計書: `docs/design/2026-07-17-reqd-plugin-design.md` (レビュー済み・承認済み)。

**Tech Stack:** Claude Code plugin (plugin.json / marketplace.json)、GitHub Actions (validate CI)、gh CLI (Issue 取得)、jq (JSON 検証)。

## Global Constraints

- 資産本文は日本語、SKILL.md frontmatter の name/description は英語、コミットメッセージは英語 (兄弟プラグイン stdd と同一の慣行)
- 運用ルール3: commands/ または skills/ を変更するコミットは、同一コミットで README インデックス + CHANGELOG + plugin.json の version を更新する (CI が強制する)
- 出典必須: skills の references・科学的根拠セクションに書く出典は、掲載前に WebSearch で実在と内容を検証する。検証できなかった出典・数値は掲載しない (stdd の掲載ルールと同一の不変条件)
- plugin.json に dependencies を持たない (スタンドアロン構成。他プラグインへはポインタ参照のみ)
- 出力パス規約: `docs/requirements/YYYY-MM-DD-<slug>.md`、frontmatter は `date` のみ
- severity 語彙は high / mid / low で統一 (high = 実装が分岐する / mid = 検証・テスト不能 / low = 表現改善)
- サイレント上書きの禁止: 既存ファイルへの書き込みは差分提示 + ユーザー確認を経る
- コミットメッセージ末尾に `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>` を付ける
- 作業ディレクトリ: `/Users/nakamurakohki/workspace/private_dev/reqd`

---

### Task 1: リポジトリ初期化 + プラグイン骨格 (v0.1.0)

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`
- Create: `.github/workflows/validate.yml`
- Create: `LICENSE`
- Create: `.gitignore`
- Create: `README.md`
- Create: `CHANGELOG.md`
- Commit (既存): `docs/design/2026-07-17-reqd-plugin-design.md`

**Interfaces:**
- Consumes: なし (最初のタスク)
- Produces: plugin name `reqd`、version `0.1.0` (後続タスクが bump する)、README のインデックス表 (後続タスクが行を追加する)、CHANGELOG (後続タスクがエントリを先頭追加する)

- [ ] **Step 1: git リポジトリを初期化する**

```bash
cd /Users/nakamurakohki/workspace/private_dev/reqd
git init -b main
```

Expected: `Initialized empty Git repository in .../reqd/.git/`

- [ ] **Step 2: plugin.json を作成する**

`.claude-plugin/plugin.json`:

```json
{
  "name": "reqd",
  "version": "0.1.0",
  "description": "Requirements definition toolkit: user story decomposition, acceptance criteria formalization (EARS / Given-When-Then), and requirements ambiguity lint",
  "author": { "name": "chun-mura" },
  "homepage": "https://github.com/chun-mura/reqd",
  "repository": "https://github.com/chun-mura/reqd",
  "license": "MIT",
  "keywords": ["requirements", "user-story", "acceptance-criteria", "ears", "ambiguity-lint"]
}
```

- [ ] **Step 3: marketplace.json を作成する**

`.claude-plugin/marketplace.json`:

```json
{
  "name": "reqd",
  "description": "Marketplace for the reqd requirements definition toolkit",
  "owner": { "name": "chun-mura" },
  "plugins": [
    {
      "name": "reqd",
      "source": ".",
      "description": "Requirements definition toolkit: user story decomposition, acceptance criteria formalization (EARS / Given-When-Then), and requirements ambiguity lint",
      "category": "workflow",
      "tags": ["requirements", "user-story", "ears", "given-when-then", "ambiguity-lint"]
    }
  ]
}
```

- [ ] **Step 4: validate CI を作成する (stdd/uidd と同一パターン)**

`.github/workflows/validate.yml`:

```yaml
name: validate

on:
  push:
    branches: [main]
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Validate plugin manifest JSON
        run: |
          jq -e . .claude-plugin/plugin.json > /dev/null
          jq -e . .claude-plugin/marketplace.json > /dev/null

      # Operating rule 3: asset changes must ship with a version bump
      - name: Require version bump when commands/skills change
        run: |
          if [ "${{ github.event_name }}" = "pull_request" ]; then
            base="origin/${{ github.base_ref }}"
          elif [ "${{ github.event.before }}" != "0000000000000000000000000000000000000000" ] \
            && git cat-file -e "${{ github.event.before }}" 2>/dev/null; then
            base="${{ github.event.before }}"
          else
            echo "No base commit to diff against; skipping bump check"
            exit 0
          fi
          changed=$(git diff --name-only "$base"...HEAD)
          if echo "$changed" | grep -qE '^(commands/|skills/)'; then
            if git diff "$base"...HEAD -- .claude-plugin/plugin.json | grep -q '"version"'; then
              echo "OK: asset change is accompanied by a version bump"
            else
              echo "commands/ or skills/ changed without a version bump in .claude-plugin/plugin.json (operating rule 3)" >&2
              exit 1
            fi
          else
            echo "No asset changes; bump not required"
          fi
          echo "changed<<EOF" >> "$GITHUB_ENV"
          echo "$changed" >> "$GITHUB_ENV"
          echo "EOF" >> "$GITHUB_ENV"

      # Operating rule 3: asset changes must ship with README index + CHANGELOG updates too
      - name: Require README/CHANGELOG update when commands/skills change
        run: |
          if echo "$changed" | grep -qE '^(commands/|skills/)'; then
            missing=""
            echo "$changed" | grep -q '^README\.md$' || missing="$missing README.md"
            echo "$changed" | grep -q '^CHANGELOG\.md$' || missing="$missing CHANGELOG.md"
            if [ -n "$missing" ]; then
              echo "commands/ or skills/ changed without updating:$missing (operating rule 3: README index + CHANGELOG + version must update together)" >&2
              exit 1
            fi
            echo "OK: asset change is accompanied by README/CHANGELOG updates"
          else
            echo "No asset changes; README/CHANGELOG update not required"
          fi
```

- [ ] **Step 5: LICENSE (MIT) と .gitignore を作成する**

`LICENSE`:

```
MIT License

Copyright (c) 2026 chun-mura

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

`.gitignore`:

```
.DS_Store
.claude/settings.local.json
```

- [ ] **Step 6: README 骨格を作成する**

`README.md`:

```markdown
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

### Skills (自動トリガする知見)

| Skill | いつ発動するか |
|---|---|

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
```

注: インデックスの2つの表はこの時点ではヘッダ行のみ (資産は未収録)。Task 2 以降で行を追加する。

- [ ] **Step 7: CHANGELOG を作成する**

`CHANGELOG.md`:

```markdown
# Changelog

## 0.1.0 - 2026-07-17

初期スキャフォールド。プラグイン骨格 (plugin.json / marketplace.json / README / CI / LICENSE) と設計書のみで資産は未収録。

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`: プラグイン定義 (dependencies なしのスタンドアロン構成、運用ルール4に基づき superpowers/aidd/estimate/stdd へはポインタ参照のみ)
- `.github/workflows/validate.yml`: stdd/uidd と同じ検証 CI (JSON 構文検証 + version bump 強制 + README/CHANGELOG 更新強制)
- `README.md`: 棲み分け定義・運用ルール5原則・併用推奨を記載
- `docs/design/2026-07-17-reqd-plugin-design.md`: 設計書 (aidd:design-review 済み)
```

- [ ] **Step 8: JSON 構文を検証する**

```bash
jq -e . .claude-plugin/plugin.json > /dev/null && jq -e . .claude-plugin/marketplace.json > /dev/null && echo OK
```

Expected: `OK`。あわせて `claude plugin validate . 2>/dev/null` が使えれば実行し、エラーがないことを確認 (コマンド不在なら jq 確認のみで可)。

- [ ] **Step 9: コミットする**

```bash
git add -A
git commit -m "chore: scaffold reqd plugin (manifest, CI, README, design doc)" -m "Standalone requirements-definition plugin skeleton following stdd/uidd conventions. Assets land in later commits per operating rule 3." -m "Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

Expected: コミット成功。`git status` がクリーン。

---

### Task 2: skills/requirements-notation (v0.2.0)

**Files:**
- Create: `skills/requirements-notation/SKILL.md`
- Create: `skills/requirements-notation/references/ears.md`
- Create: `skills/requirements-notation/references/given-when-then.md`
- Create: `skills/requirements-notation/references/user-story.md`
- Modify: `README.md` (Skills インデックスに1行追加)
- Modify: `CHANGELOG.md` (0.2.0 エントリを先頭に追加)
- Modify: `.claude-plugin/plugin.json` (version を 0.2.0 に)

**Interfaces:**
- Consumes: Task 1 の README インデックス表・CHANGELOG・plugin.json
- Produces: skill パス `skills/requirements-notation/SKILL.md` と references 3ファイル (Task 4 の `commands/new.md` がこのパスを skill フォールバックとして参照する)。記法選択フロー: 「機能単位 → ユーザーストーリー / システム挙動 → EARS / 合否基準 → Given-When-Then」

- [ ] **Step 1: 出典を WebSearch で検証する (RED に相当する事前ゲート)**

以下5件を WebSearch で検索し、実在・書誌情報 (著者・年・掲載先) が確認できるかを記録する:

1. Mavin, A., Wilkinson, P., Harwood, A., Novak, M. (2009). "Easy Approach to Requirements Syntax (EARS)". 17th IEEE International Requirements Engineering Conference (RE'09)
2. Mavin, A., Wilkinson, P. (2010). "Big Ears (The Return of Easy Approach to Requirements Engineering)". RE'10
3. North, D. (2006). "Introducing BDD". (https://dannorth.net/introducing-bdd/)
4. Wake, B. (2003). "INVEST in Good Stories, and SMART Tasks". (https://xp123.com/articles/invest-in-good-stories-and-smart-tasks/)
5. Cohn, M. (2004). "User Stories Applied: For Agile Software Development". Addison-Wesley

判定ルール: 確認できた出典のみ Step 2-4 のファイルに掲載する。書誌情報 (ページ番号・年・掲載誌) が検索結果と食い違ったら検索結果に合わせて修正する。**確認できなかった出典は掲載せず、その出典に依拠する記述 (効果の主張など) も削除する** (掲載ルールの不変条件)。Gherkin 構文の出典として Cucumber 公式ドキュメント (https://cucumber.io/docs/gherkin/reference) の存在も確認する。

Expected: 5件とも確認可能 (いずれも著名文献)。確認できない場合のみ該当記述を落とす。

- [ ] **Step 2: references/ears.md を作成する**

`skills/requirements-notation/references/ears.md`:

```markdown
# EARS (Easy Approach to Requirements Syntax)

いつ使うか: システムの応答・挙動要件 (トリガ・状態・オプション機能・望ましくない事象への応答) を1文で厳密に書くとき。

## 5つのパターン

| パターン | テンプレート | 使いどころ |
|---|---|---|
| Ubiquitous (常時) | The <system> shall <response> | 常に成り立つ性質 |
| Event-driven | When <trigger>, the <system> shall <response> | イベント起点の応答 |
| State-driven | While <state>, the <system> shall <response> | 特定状態中の挙動 |
| Optional feature | Where <feature is included>, the <system> shall <response> | 構成依存の機能 |
| Unwanted behaviour | If <unwanted condition>, then the <system> shall <response> | エラー・異常系への応答 |

複合パターン: `While <state>, when <trigger>, the <system> shall <response>` のように組み合わせてよい。日本語では「<状態> において <トリガ> のとき、<システム> は <応答> しなければならない」の形で書き、状態/トリガ/応答の要素を落とさない。

## 導出手順

1. 要件文からトリガ (イベント)・状態・望ましくない条件を特定する
2. 該当パターンを選ぶ。異常系は必ず Unwanted behaviour として正常系と別文に分ける
3. <system> と <response> を具体名で埋める。応答は観測可能・検証可能な表現にする (「適切に処理する」は不可)
4. 1文1要件 (singular)。「かつ」「また」で複数の応答を繋げない

## 科学的根拠

- Mavin, A., Wilkinson, P., Harwood, A., Novak, M. (2009). "Easy Approach to Requirements Syntax (EARS)". Proc. 17th IEEE International Requirements Engineering Conference (RE'09). — 原典。Rolls-Royce のジェットエンジン制御要件への適用で、曖昧さ・重複・過剰な複雑さの減少を報告
- Mavin, A., Wilkinson, P. (2010). "Big Ears (The Return of Easy Approach to Requirements Engineering)". Proc. 18th IEEE International Requirements Engineering Conference (RE'10). — 適用の追試報告
```

注: Step 1 の検証結果と書誌情報が食い違う場合はこの内容を検証結果に合わせて修正してから保存する (以降の references も同様)。

- [ ] **Step 3: references/given-when-then.md を作成する**

`skills/requirements-notation/references/given-when-then.md`:

```markdown
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

- North, D. (2006). "Introducing BDD". Better Software Magazine. https://dannorth.net/introducing-bdd/ — Given-When-Then テンプレートの原典
- Gherkin Reference. Cucumber 公式ドキュメント. https://cucumber.io/docs/gherkin/reference — Given/When/Then 構文の仕様
```

- [ ] **Step 4: references/user-story.md を作成する**

`skills/requirements-notation/references/user-story.md`:

```markdown
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

- Wake, B. (2003). "INVEST in Good Stories, and SMART Tasks". https://xp123.com/articles/invest-in-good-stories-and-smart-tasks/ — INVEST 基準の原典
- Cohn, M. (2004). "User Stories Applied: For Agile Software Development". Addison-Wesley. — ユーザーストーリー形式と受け入れ基準の体系化
```

- [ ] **Step 5: SKILL.md を作成する**

`skills/requirements-notation/SKILL.md`:

```markdown
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
```

- [ ] **Step 6: README インデックス・CHANGELOG・version を同時更新する**

`README.md` の Skills 表に追加:

```markdown
| `requirements-notation` | 要件・受け入れ基準を書く前。ユーザーストーリー / EARS / Given-When-Then の選択フローと記述ルールで自由散文を形式化する |
```

`CHANGELOG.md` の先頭 (`# Changelog` の直後) に追加:

```markdown
## 0.2.0 - 2026-07-17

要件記法の中核 skill を追加。

- `skills/requirements-notation/`: 記法選択フロー (SKILL.md) + 記法カタログ references 3ファイル (ユーザーストーリー・INVEST / EARS / Given-When-Then)。全記法に検証済み出典付き (Mavin et al. RE'09 / North 2006 / Wake 2003 / Cohn 2004)
- `README.md`: Skills インデックスに requirements-notation を追加
```

`.claude-plugin/plugin.json` の `"version": "0.1.0"` → `"version": "0.2.0"`

- [ ] **Step 7: 検証してコミットする**

```bash
jq -e '.version == "0.2.0"' .claude-plugin/plugin.json > /dev/null && echo OK
git add -A
git commit -m "feat: add requirements-notation skill (user story / EARS / GWT)" -m "Notation selection flow with verified sources, following stdd's citation-required invariant." -m "Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

Expected: `OK` → コミット成功。

---

### Task 3: skills/ambiguity-lint (v0.3.0)

**Files:**
- Create: `skills/ambiguity-lint/SKILL.md`
- Modify: `README.md` (Skills インデックスに1行追加)
- Modify: `CHANGELOG.md` (0.3.0 エントリを先頭に追加)
- Modify: `.claude-plugin/plugin.json` (version を 0.3.0 に)

**Interfaces:**
- Consumes: Task 1 の README/CHANGELOG/plugin.json
- Produces: skill パス `skills/ambiguity-lint/SKILL.md`。観点カタログ (観点 1〜8)、severity 定義 (high/mid/low)、指摘形式 (severity / 引用 / 観点# / 修正案)。Task 4・5 のコマンドがこの skill を参照する

- [ ] **Step 1: 出典を WebSearch で検証する**

以下3件を WebSearch で検索し、実在・書誌情報を確認する:

1. ISO/IEC/IEEE 29148:2018 "Systems and software engineering — Life cycle processes — Requirements engineering" (要件の品質特性: unambiguous / verifiable / singular 等が定義されていること)
2. Berry, D.M., Kamsties, E., Krieger, M.M. (2003). "From Contract Drafting to Software Specification: Linguistic Sources of Ambiguity — A Handbook". University of Waterloo
3. Wilson, W.M., Rosenberg, L.H., Hyatt, L.E. (1997). "Automated Analysis of Requirement Specifications" (ICSE '97 / NASA SATC。weak phrases のカタログを含むこと)

判定ルール: Task 2 Step 1 と同じ。確認できなかった出典は掲載せず、依拠する記述も削除する。特に観点1の weak words リストは Wilson et al. が確認できなければ「出典」欄から外し、ISO 29148 の unambiguous 特性のみを根拠にする。

- [ ] **Step 2: SKILL.md を作成する**

`skills/ambiguity-lint/SKILL.md`:

```markdown
---
name: ambiguity-lint
description: Use when reviewing or writing requirements, user stories, GitHub issues, or acceptance criteria — detects ambiguous vocabulary, unverifiable criteria, missing actors, undefined terms, and contradictions before they reach design, estimation, or implementation.
---

# ambiguity-lint

要件テキストの曖昧さを観点カタログで機械的に検査する。/reqd:lint の検査基準と /reqd:new の自己チェックが共用するシングルソース。

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
```

注: Step 1 で確認できなかった出典があれば、この内容から該当出典と依拠記述を削除してから保存する。

- [ ] **Step 3: README インデックス・CHANGELOG・version を同時更新する**

`README.md` の Skills 表に追加:

```markdown
| `ambiguity-lint` | 要件・Issue・受け入れ基準を書くとき/レビューするとき。曖昧語彙・検証不能な基準・アクター欠落・未定義用語・矛盾を観点カタログで検出する |
```

`CHANGELOG.md` の先頭に追加:

```markdown
## 0.3.0 - 2026-07-17

曖昧さ検査の基準 skill を追加。

- `skills/ambiguity-lint/`: 観点カタログ8項目 + severity 判定 (high/mid/low) + 指摘形式。/reqd:lint と /reqd:new 自己チェックのシングルソース。検証済み出典付き (ISO/IEC/IEEE 29148:2018 / Berry et al. 2003 / Wilson et al. 1997)
- `README.md`: Skills インデックスに ambiguity-lint を追加
```

`.claude-plugin/plugin.json` の version → `"0.3.0"`

- [ ] **Step 4: 検証してコミットする**

```bash
jq -e '.version == "0.3.0"' .claude-plugin/plugin.json > /dev/null && echo OK
git add -A
git commit -m "feat: add ambiguity-lint skill (8-perspective catalog with severity)" -m "Single source for /reqd:lint checks and /reqd:new self-check, with verified sources." -m "Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

Expected: `OK` → コミット成功。

---

### Task 4: commands/new.md (v0.4.0)

**Files:**
- Create: `commands/new.md`
- Modify: `README.md` (Commands インデックスに1行追加)
- Modify: `CHANGELOG.md` (0.4.0 エントリを先頭に追加)
- Modify: `.claude-plugin/plugin.json` (version を 0.4.0 に)

**Interfaces:**
- Consumes: `skills/requirements-notation/SKILL.md` (記法選択フロー)、`skills/ambiguity-lint/SKILL.md` (自己チェック基準・修正案の規律)。パス・観点#・severity 語彙は Task 2/3 の定義と一致させること
- Produces: `/reqd:new` コマンド。出力規約 `docs/requirements/YYYY-MM-DD-<slug>.md` (Task 5 の案内文・Task 6 のスモークが参照)

- [ ] **Step 1: commands/new.md を作成する**

`commands/new.md`:

```markdown
---
description: メモ・GitHub Issue・自由記述から要件定義書 (ユーザーストーリー + 受け入れ基準) を生成し docs/requirements/ に保存する
argument-hint: [メモのパス / Issue番号・URL / 自由記述。混在可]
---

**skill フォールバック**: requirements-notation / ambiguity-lint skill が自動発動しない場合は `skills/requirements-notation/SKILL.md` (references 含む) と `skills/ambiguity-lint/SKILL.md` を読んでから進める。

**入力解決**: $ARGUMENTS を解釈する。
- ファイルパス → 読み込む。存在しない・読み取り不可の場合は空扱いで続行せず、その旨を指摘して正しいパスの再指定かテキスト貼り付けを促す。複数入力の一部だけ読めない場合は、読めなかった事実を「未解決事項」に記録して残りで続行する
- Issue 番号 (#123 形式) / GitHub URL → `gh issue view <番号> --json title,body,comments` で取得。gh 未インストール・未認証・Issue 不存在の場合は指摘して中断せず、本文の貼り付けを促す
- 空 → 何も生成せず、入力候補 (メモのパス / Issue 番号 / 貼り付けテキスト) を案内して終了する

**ストーリー分解**: 入力を INVEST 基準 (requirements-notation の `references/user-story.md`) を満たすユーザーストーリーに分解する。

**受け入れ基準の形式化**: 各ストーリーに受け入れ基準を付与する。記法は requirements-notation の選択フローに従う (システム挙動の制約 → EARS、シナリオの合否 → Given-When-Then)。**入力から導出できない項目 (数値閾値・用語定義・優先度など) は勝手に埋めず「未解決事項」に質問形式で列挙する。**

**自己チェック**: 生成した文書を ambiguity-lint の観点カタログ (観点1〜8) で検査する。振り分けルール: 意味を変えない書き換えで直せるもの (曖昧語彙の言い換え・受動態→能動態・文脈から一意に特定できる主語補完) は修正する。数値・用語定義など入力にない情報の補完が必要なものは常に「未解決事項」へ送る。

**出力**: `docs/requirements/YYYY-MM-DD-<slug>.md` に保存 (日付は `date +%F` で取得、slug は対象の英語ケバブケース要約)。ディレクトリが無ければ作成する。

書き込み前に対象パスの存在を確認する:
1. 存在しない → そのまま保存
2. 存在する → サイレント上書きせず、既存内容との差分を提示して上書き可否を確認する。拒否された場合は `-2` 等のサフィックス付き別名で保存する
3. 改訂ポリシー: 新規生成が既定。既存要件定義書の改訂はユーザーの手動編集で行い、置き換えを明示したい場合のみ旧ファイルの frontmatter に `superseded-by: <新ファイル名>` を手動付与する

文書構成:
- frontmatter: `date` のみ
- 背景・目的
- ユーザーストーリー: 各項目 As a / I want / So that
- 受け入れ基準: ストーリーごとに EARS または Given-When-Then
- スコープ外
- 未解決事項 (open_questions): 実装が分岐する不明点を質問形式で
- 用語定義

**次のステップ案内** (プラグインの検出は行わず常時表示する非強制の案内。未インストールでも壊れない): 設計に進むなら `/aidd:design-doc <出力パス>`、見積るなら `/estimate:new <出力パス>`、受け入れ基準のテスト化は stdd の `/stdd:test-design`。未解決事項が残っている場合は先に解消を促す。
```

- [ ] **Step 2: README インデックス・CHANGELOG・version を同時更新する**

`README.md` の Commands 表に追加:

```markdown
| `/reqd:new` | メモ・GitHub Issue・自由記述から要件定義書 (ユーザーストーリー + 受け入れ基準) を生成し `docs/requirements/` に保存する |
```

`CHANGELOG.md` の先頭に追加:

```markdown
## 0.4.0 - 2026-07-17

要件定義書の生成 command を追加。

- `commands/new.md`: `/reqd:new` — 入力 (ファイル / Issue / 自由記述) をユーザーストーリーに分解し、受け入れ基準を EARS / Given-When-Then で形式化して `docs/requirements/YYYY-MM-DD-<slug>.md` に生成する。入力から導出できない項目は open_questions へ (黙認的補完の禁止)。サイレント上書き禁止 (存在チェック + 差分提示 + 確認)
- `README.md`: Commands インデックスに /reqd:new を追加
```

`.claude-plugin/plugin.json` の version → `"0.4.0"`

- [ ] **Step 3: 参照整合を検証してコミットする**

```bash
grep -q 'skills/requirements-notation/SKILL.md' commands/new.md && grep -q 'skills/ambiguity-lint/SKILL.md' commands/new.md && echo REF_OK
jq -e '.version == "0.4.0"' .claude-plugin/plugin.json > /dev/null && echo VER_OK
git add -A
git commit -m "feat: add /reqd:new command (requirements doc generation)" -m "Decomposes input into INVEST stories with EARS/GWT acceptance criteria; underivable items go to open_questions instead of silent completion." -m "Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

Expected: `REF_OK`、`VER_OK` → コミット成功。

---

### Task 5: commands/lint.md (v0.5.0)

**Files:**
- Create: `commands/lint.md`
- Modify: `README.md` (Commands インデックスに1行追加)
- Modify: `CHANGELOG.md` (0.5.0 エントリを先頭に追加)
- Modify: `.claude-plugin/plugin.json` (version を 0.5.0 に)

**Interfaces:**
- Consumes: `skills/ambiguity-lint/SKILL.md` (観点カタログ・severity・指摘形式)。severity 語彙 (high/mid/low) と観点# は Task 3 の定義と一致させること
- Produces: `/reqd:lint` コマンド (Task 6 のスモークが参照)

- [ ] **Step 1: commands/lint.md を作成する**

`commands/lint.md`:

```markdown
---
description: 要件定義書・GitHub Issue・貼り付けテキストの曖昧さを検査し、severity 付き指摘と修正案を返す
argument-hint: [ファイルパス / Issue番号・URL / 貼り付けテキスト]
---

**skill フォールバック**: ambiguity-lint skill が自動発動しない場合は `skills/ambiguity-lint/SKILL.md` を読んでから進める。

**入力解決**: $ARGUMENTS を解釈する。
- ファイルパス → 読み込む。存在しない・読み取り不可の場合は空扱いで続行せず、その旨を指摘して正しいパスの再指定かテキスト貼り付けを促す
- Issue 番号 (#123 形式) / GitHub URL → `gh issue view <番号> --json title,body,comments` で取得。gh 未インストール・未認証・Issue 不存在の場合は指摘して中断せず、本文の貼り付けを促す
- 空 → 何も検査せず、入力候補 (ファイルパス / Issue 番号 / 貼り付けテキスト) を案内して終了する

**検査**: ambiguity-lint の観点カタログ (観点1〜8) をすべて適用する。検出ごとに以下を記録する:
- severity (high = 実装が分岐する / mid = 検証・テスト不能 / low = 表現改善)
- 該当箇所の引用
- 観点# (どの観点で検出したか)
- 修正案: 意味を変えない書き換えは具体的な修正文を提案する。入力にない情報の補完が必要な場合は値を発明せず「決めるべき事項」として質問形式で出す

**出力**: severity 降順 (high → mid → low) の指摘リストを表示する。**対象ファイルは変更しない (報告のみ)。**

対象が Issue の場合は追加で「質問リスト」を出力する: 実装が分岐する不明点 (high 相当) のみを質問化し、trivial な選択は sensible default を提案して質問にしない。Issue への needs-info コメントにそのまま貼れる形式にする。

**次のステップ案内** (非強制): 指摘を反映した後、要件定義書化するなら `/reqd:new <対象>`。既に要件定義書なら修正後に設計 (`/aidd:design-doc`) へ。
```

- [ ] **Step 2: README インデックス・CHANGELOG・version を同時更新する**

`README.md` の Commands 表に追加:

```markdown
| `/reqd:lint` | 要件定義書・GitHub Issue・貼り付けテキストの曖昧さを検査し、severity 付き指摘と修正案 (Issue の場合は needs-info 質問リスト) を返す |
```

`CHANGELOG.md` の先頭に追加:

```markdown
## 0.5.0 - 2026-07-17

曖昧さ検査 command を追加。

- `commands/lint.md`: `/reqd:lint` — ambiguity-lint の観点カタログで検査し、severity 降順の指摘リスト + 修正案を返す。対象が Issue の場合は needs-info 質問リスト (aidd-autopilot triage の questions と同形) も出力する。対象ファイルは変更しない (報告のみ)
- `README.md`: Commands インデックスに /reqd:lint を追加
```

`.claude-plugin/plugin.json` の version → `"0.5.0"`

- [ ] **Step 3: 参照整合を検証してコミットする**

```bash
grep -q 'skills/ambiguity-lint/SKILL.md' commands/lint.md && echo REF_OK
jq -e '.version == "0.5.0"' .claude-plugin/plugin.json > /dev/null && echo VER_OK
git add -A
git commit -m "feat: add /reqd:lint command (ambiguity check with severity)" -m "Report-only lint over files, GitHub issues, and pasted text; emits needs-info question list for issues." -m "Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

Expected: `REF_OK`、`VER_OK` → コミット成功。

---

### Task 6: スモーク検証 (設計書「テスト/検証」節の3本)

**Files:**
- Create (作業用、リポジトリ外): スクラッチディレクトリ `<scratchpad>/reqd-smoke/` とフィクスチャ2ファイル
- Modify (指摘があった場合のみ): `commands/new.md`, `commands/lint.md`, `skills/*` + README/CHANGELOG/version 同時更新

**Interfaces:**
- Consumes: Task 1〜5 の全成果物 (コマンド2本・skill 2本)
- Produces: 検証済みプラグイン。修正が入った場合は v0.5.1 (資産の修正のみなら patch bump)

- [ ] **Step 1: フィクスチャを作成する**

スクラッチディレクトリを作り、既知の曖昧さを仕込んだメモを置く。

`<scratchpad>/reqd-smoke/memo.md`:

```markdown
# 通知機能のメモ

会員がコメントを受け取ったら通知したい。通知はメールなどで送信される。
管理者は必要に応じて通知設定を適切に変更できる。
通知画面は高速に表示されること。既読管理もできると良い。上限件数は TBD。
```

仕込んだ曖昧さ (期待検出リスト): 「など」(観点1) / 「必要に応じて」「適切に」(観点1) / 「高速に」(観点2) / 「送信される」の主体不明 (観点3) / 「会員」と後続の「ユーザー」揺れがあれば観点4 / 「TBD」(観点7) / 「できると良い」の要否不明。

- [ ] **Step 2: スモーク1 — /reqd:new の手順を実行する**

`<scratchpad>/reqd-smoke/` を作業ディレクトリとして、`commands/new.md` の手順を memo.md を入力に忠実に実行する (subagent に new.md 全文とフィクスチャパスを渡して実行させる。プラグイン未インストール環境でも command ファイルの手順を直接実行すれば検証できる)。

検証チェックリスト:
- [ ] 出力が `docs/requirements/YYYY-MM-DD-<slug>.md` に生成された (ディレクトリ自動作成)
- [ ] frontmatter が `date` のみ
- [ ] 7セクション (背景・目的 / ユーザーストーリー / 受け入れ基準 / スコープ外 / 未解決事項 / 用語定義) が揃っている
- [ ] ユーザーストーリーが As a / I want / So that 形式
- [ ] 受け入れ基準が EARS または GWT 形式 (自由散文でない)
- [ ] 「上限件数」「高速の閾値」「通知の送信主体」「既読管理の要否」が**発明されず**未解決事項に質問形式で入っている
- [ ] 末尾に /aidd:design-doc・/estimate:new への案内がある

- [ ] **Step 3: スモーク2 — /reqd:lint の手順を実行する**

同じフィクスチャ memo.md に対して `commands/lint.md` の手順を実行する。

検証チェックリスト:
- [ ] Step 1 の期待検出リストのうち「など」「必要に応じて」「適切に」「高速に」「送信される (主体不明)」「TBD」が検出された
- [ ] 各指摘に severity / 引用 / 観点# / 修正案が付いている
- [ ] severity 降順に並んでいる
- [ ] memo.md 自体は変更されていない
- [ ] Issue 扱いで再実行した場合、needs-info 質問リストが出力され、trivial な選択が質問化されていない

- [ ] **Step 4: スモーク3 — 下流連携を確認する**

スモーク1の出力ファイルを下流に渡して受理されることを確認する:

```
/aidd:design-doc <スモーク1の出力パス>
/estimate:new <スモーク1の出力パス>
```

検証チェックリスト:
- [ ] /aidd:design-doc が要件ファイルとして読み込み、設計書生成に進める (入力形式の不整合で止まらない)
- [ ] /estimate:new の spec-analyzer が change_units を抽出でき、要件定義書の「未解決事項」が open_questions に反映される

(いずれかのプラグインが実行環境にない場合: そのプラグインの入力仕様 — aidd:design-doc は「要件の要約 or 要件ファイルのパス」、estimate spec-analyzer は「file paths and/or pasted text」 — に対し、出力ファイルの構造が適合していることを机上確認し、未実施の連携はその旨を記録する)

- [ ] **Step 5: 指摘を修正してコミットする**

スモークで発覚した資産の不備 (テンプレートの欠落、検出漏れを生む観点記述の曖昧さ等) を修正する。commands/ か skills/ に修正が入った場合は README/CHANGELOG/version (0.5.1) を同時更新する。修正がなければこのステップはスキップ。

```bash
git add -A
git commit -m "fix: address smoke-test findings in command/skill assets" -m "Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

- [ ] **Step 6: 最終検証**

```bash
jq -e . .claude-plugin/plugin.json > /dev/null && jq -e . .claude-plugin/marketplace.json > /dev/null && echo JSON_OK
git log --oneline
git status
```

Expected: `JSON_OK`、コミット履歴が Task 1〜6 分揃っている、working tree クリーン。README のインデックスに Commands 2行・Skills 2行が揃っていることを目視確認する。

---

## Self-Review 結果 (計画作成時に実施済み)

- **Spec coverage**: 設計書の全セクション (スコープ3機能 / 棲み分け / ディレクトリ構造 / new・lint・2 skill の仕様 / 出力パスライフサイクル / エラーハンドリング / CI / スモーク3本 / 運用ルール / 実装順序) に対応するタスクあり。ギャップなし
- **Placeholder scan**: 全ファイル内容を実体で記載。TBD なし (フィクスチャ内の「TBD」は検出対象として意図的)
- **Type consistency**: skill パス (`skills/requirements-notation/SKILL.md`, `skills/ambiguity-lint/SKILL.md`)、severity 語彙 (high/mid/low)、観点# (1〜8)、出力規約 (`docs/requirements/YYYY-MM-DD-<slug>.md`) を全タスクで統一済み
