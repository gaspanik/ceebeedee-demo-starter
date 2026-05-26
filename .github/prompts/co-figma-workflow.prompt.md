---
description: Figma URLを受け取り、実装→レビュー→最適化の3フェーズを順番に実行する
tools:
  - search/codebase
  - edit/editFiles
  - execute/getTerminalOutput
  - execute/runInTerminal
  - com.figma.mcp/mcp/get_design_context
  - com.figma.mcp/mcp/get_variable_defs
  - com.figma.mcp/mcp/get_screenshot
---

> **前提条件**: このプロンプトは単体では動作しません。同じ `.github/prompts/` フォルダ内に `co-implement-figma.prompt.md`・`co-review-figma.prompt.md`・`co-code-optim.prompt.md` の3ファイルが存在すること。存在しない場合はユーザーに通知して処理を中止すること。

実装する **単一の** Figma URL をチャットで指定してください。複数URLが渡された場合は1つずつ順番に処理します。URLが無効・アクセス不可・`get_design_context` がエラーを返した場合は、エラー内容を表示して処理を中止し、有効なURLの再入力を求めること。

> **注意**: ターミナルコマンドを実行する際は **Bash** を使用すること。パッケージマネージャーはプロジェクトルートの lockfile を確認して自動判定すること（`package-lock.json` → npm、`yarn.lock` → yarn、`pnpm-lock.yaml` → pnpm）。

以下の3フェーズを **必ず Phase 1 → Phase 2 → Phase 3 の順に** 実行する。各フェーズの完了確認に記載された **全項目を出力し終えた時点** でそのフェーズは完了とみなし、次のフェーズに進む。並列実行は不可。

---

## Phase 1: 実装

[co-implement-figma.prompt.md](co-implement-figma.prompt.md) の手順に従って実装を行う。

**Phase 1 中に Figma API エラー・ファイル書き込みエラー・コンパイルエラーが発生した場合は、即座に処理を中止し、エラー内容と原因をユーザーに報告すること。Phase 2 には進まないこと。**

Phase 1 完了確認（全項目を出力すること）:
- 作成・変更したファイルの一覧
- 新たに `src/index.css` の `@theme` ブロックに追加したデザイントークン（色・スペーシング・フォントなど）があれば列挙する（なければ「なし」と記載）
- `src/assets/images/` にダウンロードした画像ファイルの一覧（なければ「なし」と記載）

上記を全て出力したら「--- Phase 1 完了 ---」と出力し、Phase 2 に進む。

---

## Phase 2: レビューと修正

[co-review-figma.prompt.md](co-review-figma.prompt.md) の手順に従ってレビュー・修正を行う。

TypeScript / Biome エラーは最大3回の修正を試みること。3回試みてもエラーが解消されない場合は、そのエラーを残課題として記録し Phase 3 に進む。自動修正不可能なエラーを無限に試みないこと。

Phase 2 完了確認（全項目を出力すること）:
- 視覚的差異として修正した箇所
- TypeScript / Biome のエラー解消状況（全解消 or 残エラーの内容）
- 未解決の残課題を以下のカテゴリで明記する: (1) 視覚的差異, (2) TypeScript/Biome エラー, (3) アクセシビリティ問題, (4) レスポンシブ対応漏れ（各カテゴリで課題がなければ「なし」と記載）

上記を全て出力したら「--- Phase 2 完了 ---」と出力し、Phase 3 に進む。

---

## Phase 3: 最適化

[co-code-optim.prompt.md](co-code-optim.prompt.md) の手順に従ってリファクタリングを行う。

リファクタリング後に `pnpm build` と `pnpm check`（またはプロジェクトのパッケージマネージャーに合わせたコマンド）を実行し、新たなエラーが発生した場合は該当の変更を元に戻してから原因を報告すること。

Phase 3 完了確認（全項目を出力すること）:
- リファクタリングしたコンポーネントと変更内容
- `src/index.css` の `@theme` ブロックに追加・整理したトークン（なければ「なし」と記載）
- 選択したコンポーネントパターン（`cn` / CVA / tailwind-variants）とその理由

上記を全て出力したら「--- Phase 3 完了 ---」と出力する。

---

## 全フェーズ完了後

各フェーズの完了確認時に出力した内容を **そのまま引用** して以下のサマリーを構成すること。追加の推論や再評価は行わない。

1. **Phase 1 実装サマリー** — 作成ファイル・追加トークン・ダウンロード画像
2. **Phase 2 レビューサマリー** — 修正箇所・残課題（カテゴリ別）
3. **Phase 3 最適化サマリー** — リファクタリング内容・採用パターン
