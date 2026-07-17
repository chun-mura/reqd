# Changelog

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
