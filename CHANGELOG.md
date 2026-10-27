# Changelog

## 0.6.0 - 2026-10-27

commands を skills に移し、Codex と Cursor でも使えるようにした。Claude Code での呼び出し方 (`/reqd:new`・`/reqd:lint`) は変わらない。

- `skills/new/SKILL.md`, `skills/lint/SKILL.md`: `commands/new.md`・`commands/lint.md` から移動。`name` と `disable-model-invocation: true` を追加し、`${CLAUDE_PLUGIN_ROOT}/skills/...` を skill からの相対パスに変更。`$ARGUMENTS` が展開されないハーネスでの読み替えを追記
- `skills/new/agents/openai.yaml`, `skills/lint/agents/openai.yaml`: Codex でモデルから自動起動しない設定
- `.codex-plugin/plugin.json`, `.cursor-plugin/plugin.json`: Codex と Cursor 用のマニフェストを追加
- `README.md`: 導入手順と呼び出し方を3ハーネス分に、インデックスを skills に変更
- `CLAUDE.md`: commands から skills への移動を反映

## 0.5.7 - 2026-07-20

生成物 frontmatter に任意の試験フィールド `related`（自由記述）を導入。

- `commands/new.md`: frontmatter を `date` 必須 + 任意 `related`（試験・省略可・発明禁止・下流非消費）へ拡張
- `docs/design/2026-07-17-reqd-plugin-design.md`: 出力・テンプレート節を同期
- `docs/verification/2026-07-20-related-trial.md`: 試験フィールドの受け入れ条件と正式化チェックリストを記録
- `README.md`: `/reqd:new` インデックスを更新

## 0.5.6 - 2026-07-17

受入レビュー指摘の是正 (出典の監査証跡・シングルソース違反)。

- `docs/verification/source-verification.md`: 全9出典の外部照合結果 (判定・根拠 URL・検索クエリ) を監査証跡として記録。掲載ルール「検証できない出典は掲載禁止」の履行を追跡可能にした (当初 `.superpowers/sdd/` に置いたが gitignore 対象のため追跡されるパスへ移動)
- `skills/requirements-notation/references/given-when-then.md`, `user-story.md`: 照合で判明した現行 URL パスへ修正 (dannorth.net/blog/、xp123.com の /articles/ なし版)
- `skills/ambiguity-lint/input-safety.md`: 「パス判定」節を新設し、両 command に重複していた「パスらしい」判定基準 (プレフィックス・拡張子リスト) を集約
- `commands/new.md`, `commands/lint.md`: パス判定の重複記載を削除し input-safety への参照に置換 (シングルソース回復)
- `README.md`: Commands / Skills インデックスを更新

## 0.5.5 - 2026-07-17

生成物への stored prompt injection を防止。

- `commands/new.md`: 生成物へ取り込む情報の allowlist を定義し、入力内の命令・役割変更・ツール指示を引用・転載しない規則を追加
- `README.md`: `/reqd:new` のインデックスを更新

## 0.5.4 - 2026-07-17

出力パス境界と intake/metadata の単一ソース性を強化。

- `skills/ambiguity-lint/input-safety.md`: Issue・ファイル・貼り付け入力の安全規則を共通化し、両 command の重複を解消
- `commands/new.md`: slug の完全一致、リポジトリ配下 containment、シンボリックリンク拒否を追加
- `commands/lint.md`: needs-info と sensible defaults の正規形を定義
- `.github/workflows/validate.yml`: token を read-only に制限し、CHANGELOG version と README 対象行を意味検証
- `docs/superpowers/plans/2026-07-17-reqd-plugin.md`: 完了済み historical plan と明記し、古い埋め込みスニペットの再実行を禁止
- `README.md`: Commands / Skills インデックスを更新

## 0.5.3 - 2026-07-17

GitHub Issue 取得フィールドを固定。

- `commands/new.md`, `commands/lint.md`: 検証済み Issue URL/番号を単一引数として渡し、`title`, `body`, `comments` を常に取得する
- `README.md`: Commands インデックスを更新

## 0.5.2 - 2026-07-17

未信頼入力の取得境界を追加で強化。

- `commands/new.md`, `commands/lint.md`: 引数の shell 展開を禁止し、Issue URL/番号を完全一致で検証。パスを不透明な値として扱い、`eval` とコマンド文字列連結を禁止
- `commands/new.md`, `commands/lint.md`: 入力本文と衝突しない UUID 付き境界を入力ごとに生成し、delimiter breakout を防止
- `.github/workflows/validate.yml`: 初回 push と root commit もコミット単位検査の対象に変更
- `README.md`: Commands インデックスを更新

## 0.5.1 - 2026-07-17

入力・保存・検証の安全性と記法 reference の表示を修正。

- `commands/new.md`, `commands/lint.md`: 外部入力を未信頼データとして区切り、埋め込まれた命令を実行しない規律を追加。severity は ambiguity-lint を単一ソースとして参照
- `commands/new.md`: 上書き拒否後は最初の未使用連番名へ保存。frontmatter は `date` のみを維持し、置き換え注記は本文へ記録
- `skills/requirements-notation/references/`: EARS プレースホルダーの Markdown 表示と North (2006) の雑誌/Web公開日を修正
- `.github/workflows/validate.yml`: 運用ルール3を各コミット単位で検査し、資産と release metadata の別コミット化を拒否
- `README.md`: Commands / Skills インデックスを更新

## 0.5.0 - 2026-07-17

曖昧さ検査 command を追加。

- `commands/lint.md`: `/reqd:lint` — ambiguity-lint の観点カタログで検査し、severity 降順の指摘リスト + 修正案を返す。対象が Issue の場合は needs-info 質問リスト (aidd-autopilot triage の questions と同形) も出力する。対象ファイルは変更しない (報告のみ)
- `commands/lint.md`, `commands/new.md`: 入力判定の優先順位を統一し、Issue URL / `#番号` / `file:<path>` / 既存ファイルパス / 貼り付けテキストを正規化。`new` は shell 風の引用規則による混在入力と自由記述の再結合に対応し、存在しないパスらしい入力は自由文にせずエラーとして扱う
- `README.md`: Commands インデックスに /reqd:lint を追加

## 0.4.0 - 2026-07-17

要件定義書の生成 command を追加。

- `commands/new.md`: `/reqd:new` — 入力 (ファイル / Issue / 自由記述) をユーザーストーリーに分解し、受け入れ基準を EARS / Given-When-Then で形式化して `docs/requirements/YYYY-MM-DD-<slug>.md` に生成する。入力から導出できない項目は open_questions へ (黙認的補完の禁止)。サイレント上書き禁止 (存在チェック + 差分提示 + 確認)
- `README.md`: Commands インデックスに /reqd:new を追加

## 0.3.0 - 2026-07-17

曖昧さ検査の基準 skill を追加。

- `skills/ambiguity-lint/`: 観点カタログ8項目 + severity 判定 (high/mid/low) + 指摘形式。/reqd:lint と /reqd:new 自己チェックのシングルソース。検証済み出典付き (ISO/IEC/IEEE 29148:2018 / Berry et al. 2003 / Wilson et al. 1997)
- `README.md`: Skills インデックスに ambiguity-lint を追加

## 0.2.0 - 2026-07-17

要件記法の中核 skill を追加。

- `skills/requirements-notation/`: 記法選択フロー (SKILL.md) + 記法カタログ references 3ファイル (ユーザーストーリー・INVEST / EARS / Given-When-Then)。全記法に検証済み出典付き (Mavin et al. RE'09 / North 2006 / Wake 2003 / Cohn 2004)
- `README.md`: Skills インデックスに requirements-notation を追加

## 0.1.0 - 2026-07-17

初期スキャフォールド。プラグイン骨格 (plugin.json / marketplace.json / README / CI / LICENSE) と設計書のみで資産は未収録。

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`: プラグイン定義 (dependencies なしのスタンドアロン構成、運用ルール4に基づき superpowers/aidd/estimate/stdd へはポインタ参照のみ)
- `.github/workflows/validate.yml`: stdd/uidd と同じ検証 CI (JSON 構文検証 + version bump 強制 + README/CHANGELOG 更新強制)
- `README.md`: 棲み分け定義・運用ルール5原則・併用推奨を記載
- `docs/design/2026-07-17-reqd-plugin-design.md`: 設計書 (aidd:design-review 済み)
