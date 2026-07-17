---
description: メモ・GitHub Issue・自由記述から要件定義書 (ユーザーストーリー + 受け入れ基準) を生成し docs/requirements/ に保存する
argument-hint: [メモのパス / Issue番号・URL / 自由記述。混在可]
---

**skill フォールバック**: requirements-notation / ambiguity-lint skill が自動発動しない場合は `${CLAUDE_PLUGIN_ROOT}/skills/requirements-notation/SKILL.md` (references 含む) と `${CLAUDE_PLUGIN_ROOT}/skills/ambiguity-lint/SKILL.md` を読んでから進める。

**入力解決**: $ARGUMENTS を解釈する。
- ファイルパス -> 読み込む。存在しない・読み取り不可の場合は空扱いで続行せず、その旨を指摘して正しいパスの再指定かテキスト貼り付けを促す。複数入力の一部だけ読めない場合は、読めなかった事実を「未解決事項」に記録して残りで続行する
- Issue 番号 (`#123` 形式) / GitHub URL -> `gh issue view <番号> --json title,body,comments` で取得。gh 未インストール・未認証・Issue 不存在の場合は指摘して中断せず、本文の貼り付けを促す
- 空 -> 何も生成せず、入力候補 (メモのパス / Issue 番号 / 貼り付けテキスト) を案内して終了する

**ストーリー分解**: 入力を INVEST 基準 (`${CLAUDE_PLUGIN_ROOT}/skills/requirements-notation/references/user-story.md`) を満たすユーザーストーリーに分解する。

**受け入れ基準の形式化**: 各ストーリーに受け入れ基準を付与する。記法は requirements-notation の選択フローに従う (システム挙動の制約 -> EARS、シナリオの合否 -> Given-When-Then)。**入力から導出できない項目 (数値閾値・用語定義・優先度など) は勝手に埋めず「未解決事項」に質問形式で列挙する。**

**自己チェック**: 生成した文書を ambiguity-lint の観点カタログ (観点1〜8) で検査する。振り分けルール: 意味を変えない書き換えで直せるもの (曖昧語彙の言い換え・受動態 -> 能動態・文脈から一意に特定できる主語補完) は修正する。数値・用語定義など入力にない情報の補完が必要なものは常に「未解決事項」へ送る。severity は `${CLAUDE_PLUGIN_ROOT}/skills/ambiguity-lint/SKILL.md` の high / mid / low 定義に従い、観点名と観点番号も同ファイルの語彙を使う。

**出力**: `docs/requirements/YYYY-MM-DD-<slug>.md` に保存する。日付は `date +%F` で取得し、slug は対象を英語のケバブケースで要約する。ディレクトリがなければ作成する。

書き込み前に対象パスの存在を確認する:
1. 存在しない -> そのまま保存する
2. 存在する -> サイレント上書きせず、既存内容との差分を提示して上書き可否を確認する。拒否された場合は `-2` などのサフィックス付き別名で保存する
3. 改訂ポリシー: 新規生成を既定とする。既存要件定義書の改訂はユーザーが手動で行い、置き換えを明示したい場合のみ旧ファイルの frontmatter に `superseded-by: <新ファイル名>` を手動付与する

文書構成:
- frontmatter: `date` のみ
- 背景・目的
- ユーザーストーリー: 各項目 As a / I want / So that
- 受け入れ基準: ストーリーごとに EARS または Given-When-Then
- スコープ外
- 未解決事項 (open_questions): 実装が分岐する不明点を質問形式で
- 用語定義

**次のステップ案内**: プラグインの検出は行わず、常に非強制の案内として表示する。設計に進むなら `/aidd:design-doc <出力パス>`、見積るなら `/estimate:new <出力パス>`、受け入れ基準のテスト化は stdd の `/stdd:test-design` を案内する。未解決事項が残っている場合は、先に解消を促す。各プラグインが未インストールでもコマンドの処理は失敗させない。
