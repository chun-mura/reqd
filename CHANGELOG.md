# Changelog

## 0.1.0 - 2026-07-17

初期スキャフォールド。プラグイン骨格 (plugin.json / marketplace.json / README / CI / LICENSE) と設計書のみで資産は未収録。

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`: プラグイン定義 (dependencies なしのスタンドアロン構成、運用ルール4に基づき superpowers/aidd/estimate/stdd へはポインタ参照のみ)
- `.github/workflows/validate.yml`: stdd/uidd と同じ検証 CI (JSON 構文検証 + version bump 強制 + README/CHANGELOG 更新強制)
- `README.md`: 棲み分け定義・運用ルール5原則・併用推奨を記載
- `docs/design/2026-07-17-reqd-plugin-design.md`: 設計書 (aidd:design-review 済み)
