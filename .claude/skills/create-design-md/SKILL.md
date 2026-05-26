---
name: create-design-md
description: Google の design.md 仕様に沿って DESIGN.md をプロジェクトルートに生成する。コードベース探索・Figma URL・既存 DESIGN.md URL の3つのソースモードに対応し、design.md リンターで検証する。
argument-hint: "[figma-url | design-md-url]"
allowed-tools: Bash, Read, Write, Edit, Agent, AskUserQuestion, WebFetch, mcp__plugin_figma_figma__get_design_context, mcp__plugin_figma_figma__get_screenshot, mcp__plugin_figma_figma__get_metadata, mcp__plugin_figma_figma__get_variable_defs, mcp__plugin_figma_figma__search_design_system
---

# DESIGN.md を生成する

[Google design.md 仕様](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md) に沿って `DESIGN.md` をプロジェクトルートに生成し、リンターで検証する。

**返答言語:** 日本語で返答する。

## Step 1: ソースを決定する

まずプロジェクトルートに `DESIGN.md` がすでに存在するか確認する。存在する場合は、進める前に AskUserQuestion でユーザーに確認する：

- **上書きして再生成** — 新しいソースから `DESIGN.md` を再生成する（既存ファイルを上書き）
- **そのまま残す** — 中断して既存ファイルを変更しない

「そのまま残す」を選んだ場合はここで終了し、既存の `DESIGN.md` を変更しなかったことを報告する。

「上書きして再生成」を選んだ場合（または `DESIGN.md` が存在しない場合）、`$ARGUMENTS` を以下のルールで評価する：

- `figma.com` を含む URL → **Figma モード**
- `.md` で終わる URL、または `raw.githubusercontent.com` / `github.com` を含む URL → **DESIGN.md URL モード**
- 引数なし → AskUserQuestion でユーザーに確認する

確認時の選択肢：

- **コードベースを探索** — プロジェクトのコードと CSS からデザイン情報を抽出する
- **Figma URL** — Figma デザインファイルの URL を入力してもらう
- **既存の DESIGN.md URL** — 公開されている DESIGN.md を取得する（GitHub raw URL など）

## Step 2: デザイン情報を収集する

### DESIGN.md URL モード（Step 2A）

標準的な GitHub URL が指定された場合は raw URL に変換する：
- `github.com/<user>/<repo>/blob/<branch>/<path>` → `raw.githubusercontent.com/<user>/<repo>/<branch>/<path>`

`WebFetch` でファイルを取得し、プロジェクトルートに `DESIGN.md` として保存する。  
その後 **Step 4（リント検証）** に直接進む — Step 3 は不要。

### コードベースモード

以下のファイルを順番に読み込み、デザインシステムの情報を収集する：

1. `src/index.css` — `@import "tailwindcss"` と `@theme` ブロックからカラー・スペーシングトークンを抽出する
2. `src/routes/` 配下の `*.tsx` ファイル — 使用されているコンポーネントパターン（ボタン、カードなど）を確認する
3. `src/components/` 配下の `*.tsx` ファイル — 再利用コンポーネントのスタイルパターンを確認する
4. `src/main.tsx` — ルーター設定など UI に関わる実装を確認する
5. `package.json` — プロジェクト名を取得する

収集する情報：
- プロジェクト名とブランド名
- カラーパレット（16進数値）
- タイポグラフィ（フォントファミリー・サイズ・ウェイト）
- スペーシングスケール
- ボーダーラジアス
- コンポーネントのスタイルパターン（ボタンなど）
- 全体的なデザインのムードとトーン

### Figma モード

Figma URL から `fileKey` と `nodeId` を抽出する（`node-id=1-2` → `1:2` に変換）。

以下のツールでデザイン情報を収集する：
- `get_metadata` — ファイル名とプロジェクト概要
- `get_variable_defs` — Figma 変数（カラー・タイポグラフィ・スペーシングトークン）
- `get_design_context` — 詳細なデザインコンテキストとコンポーネント情報

コードベースモードと同じ情報を収集する。

## Step 3: DESIGN.md を生成する

以下の仕様に沿って、プロジェクトルートに `DESIGN.md` を生成する。

### YAML フロントマター

```yaml
---
version: alpha
name: <プロジェクト名>
description: <1行のプロジェクト説明>
colors:
  primary: "#XXXXXX"
  secondary: "#XXXXXX"
  # tertiary, neutral, surface なども必要に応じて追加
typography:
  h1:
    fontFamily: <フォント名>
    fontSize: <px>
    fontWeight: <数値>
    lineHeight: <数値>
    letterSpacing: <em>
  body-md:
    fontFamily: <フォント名>
    fontSize: <px>
    fontWeight: <数値>
    lineHeight: <数値>
  # label-md, caption なども必要に応じて追加
rounded:
  sm: <px>
  md: <px>
  lg: <px>
  full: 9999px
spacing:
  xs: <px>
  sm: <px>
  md: <px>
  lg: <px>
  xl: <px>
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "#FFFFFF"
    rounded: "{rounded.md}"
    padding: <px>
  button-primary-hover:
    backgroundColor: "{colors.secondary}"
  # その他のコンポーネントも必要に応じて追加
---
```

### トークンルール

- **Color**: `#` + 6桁の16進数（例: `"#1A1C1E"`）— 必ずダブルクォートで囲む
- **Dimension**: 数値 + 単位（`px`、`em`、`rem`）— 例: `48px`、`-0.02em`
- **トークン参照**: `{path.to.token}` — クォート不要（例: `"{colors.primary}"`）
- **fontWeight**: 数値（例: `400`、`700`）
- **lineHeight**: 数値または Dimension（例: `1.6` または `24px`）
- `colors.primary` は必須 — 省略すると `missing-primary` 警告が出る
- コンポーネントの `backgroundColor` / `textColor` ペアは WCAG AA のコントラスト比（4.5:1）を確保する

### Markdown 本文（セクション順序は固定）

```markdown
## Overview

[ブランドパーソナリティ・ターゲットユーザー・UI が伝えるべき印象を 3〜5 文で記述]

## Colors

[カラーパレットの説明。各カラーの役割と使用場面を箇条書きで]

- **Primary (#XXXXXX):** [説明]
- **Secondary (#XXXXXX):** [説明]

## Typography

[タイポグラフィ戦略。使用フォントと各レベルの役割]

## Layout

[レイアウトとスペーシング戦略。グリッドシステムとホワイトスペースの考え方]

## Elevation & Depth

[視覚的階層の表現方法。シャドウ、トーンレイヤーなど]

## Shapes

[ボーダーラジアスと形状の言語]

## Components

[主要コンポーネントのスタイルガイドライン]

## Do's and Don'ts

- Do [推奨プラクティス]
- Don't [禁止プラクティス]
```

**注意:** 情報が不十分なセクションは省略可。ただし含めるセクションは上記の順序に従うこと。

## Step 4: リント検証

ファイル生成後にリンターを実行する。`<pm>` はこのプロジェクトで使用するパッケージマネージャー（`npx`、`pnpm dlx`、`yarn dlx` など）に置き換える：

```bash
<pm> @google/design.md lint DESIGN.md
```

結果を確認する：

- **error** → `DESIGN.md` を修正して再実行する（エラーがなくなるまで繰り返す）
- **warning** → 確認して可能な限り修正する（コントラスト比の問題、孤立トークンなど）
- **info** → 対応不要

### よくあるリントエラーと修正方法

| エラー | 原因 | 修正 |
|---|---|---|
| `broken-ref` | トークン参照 `{colors.xxx}` が存在しない | トークン名を確認して修正する |
| `missing-primary` | `colors.primary` が未定義 | `colors` セクションに `primary` を追加する |
| `contrast-ratio` | テキスト/背景のコントラスト比が 4.5:1 未満 | コントラストの高い色に調整する |
| `section-order` | セクションが正しい順序になっていない | Overview → Colors → … の順に並べ直す |

## Step 5: 完了レポートを出力する

以下を報告する：
- 生成した `DESIGN.md` のパス
- リント結果のサマリー（エラー / 警告 / 情報の件数）
- 主要デザイントークンの概要（カラー・タイポグラフィ）
