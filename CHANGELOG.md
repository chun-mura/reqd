# Changelog

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
