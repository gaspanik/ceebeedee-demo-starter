---
description: Google の design.md 仕様に沿って DESIGN.md をプロジェクトルートに生成する。コードベース探索・Figma URL・既存 DESIGN.md URL の3つのソースモードに対応する
tools:
  - search/codebase
  - edit/editFiles
  - execute/getTerminalOutput
  - execute/runInTerminal
  - com.figma.mcp/mcp/get_design_context
  - com.figma.mcp/mcp/get_screenshot
  - com.figma.mcp/mcp/get_metadata
  - com.figma.mcp/mcp/get_variable_defs
  - com.figma.mcp/mcp/search_design_system
---

[Google design.md 仕様](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md) に沿って `DESIGN.md` をプロジェクトルートに生成し、リンターで検証します。

**返答言語:** 日本語で返答する。

---

## Step 1: ソースを決定する

まずプロジェクトルートに `DESIGN.md` がすでに存在するか確認する。存在する場合はユーザーに確認する：

- **上書きして再生成** — 新しいソースから `DESIGN.md` を再生成する（既存ファイルを上書き）
- **そのまま残す** — 中断して既存ファイルを変更しない

「そのまま残す」を選んだ場合はこのプロセスを完全に終了し、既存の `DESIGN.md` を一切変更しない。再開するには改めてプロンプトを呼び出す必要がある。

続ける場合（または `DESIGN.md` が存在しない場合）、以下の表に従ってモードを決定する：

| 入力の条件 | 採用するモード |
|---|---|
| `figma.com` を含む URL | **Figma モード** |
| `.md` で終わる URL、または `raw.githubusercontent.com` / `github.com` を含む URL | **DESIGN.md URL モード** |
| URL 形式だが上記に該当しない | ユーザーに無効な URL であることを通知し、プロセスを終了する |
| 引数なし、または URL 以外の文字列 | ユーザーに以下の選択肢を確認する |

引数なし・URL 以外の場合の選択肢：

- **コードベースを探索** — プロジェクトのコードと CSS からデザイン情報を抽出する
- **Figma URL** — Figma デザインファイルの URL を入力してもらう
- **既存の DESIGN.md URL** — 公開されている DESIGN.md を取得する（GitHub raw URL など）

---

## Step 2: デザイン情報を収集する

### DESIGN.md URL モード（Step 2A）

標準的な GitHub URL が指定された場合は raw URL に変換する：
- `github.com/<user>/<repo>/blob/<branch>/<path>` → `raw.githubusercontent.com/<user>/<repo>/<branch>/<path>`

URL からファイルを取得し、プロジェクトルートに `DESIGN.md` として保存する。  
その後 **Step 4（リント検証）** に直接進む。

### コードベースモード

以下のファイルを読み込み、デザインシステムの情報を収集する：

1. `src/index.css` — `@import url(...)` フォント指定と `@theme` ブロックからカラー・タイポグラフィ・スペーシングトークンを抽出する（Tailwind CSS v4 形式）
2. `index.html` — ルート HTML からメタ情報・フォント読み込みを確認する
3. `src/main.tsx` — ルーター設定など UI に関わる実装を確認する
4. `src/routes/` — TanStack Router のページファイルから使用されているコンポーネントパターンを確認する（`__root.tsx`・`index.tsx` など）
5. `src/components/` — 既存コンポーネント（`ButtonCn.tsx` など）のスタイルパターンを確認する
6. `package.json` — プロジェクト名を取得する

収集する情報：プロジェクト名・カラーパレット・タイポグラフィ（フォントファミリー含む）・スペーシングスケール・ボーダーラジアス・コンポーネントのスタイルパターン・デザインのムードとトーン

### Figma モード

Figma URL から `fileKey` と `nodeId` を抽出する（`node-id=1-2` → `1:2` に変換）。

以下のツールでデザイン情報を収集する：
- `get_metadata` — ファイル名とプロジェクト概要
- `get_variable_defs` — Figma 変数（カラー・タイポグラフィ・スペーシングトークン）
- `get_design_context` — 詳細なデザインコンテキストとコンポーネント情報

---

## Step 3: DESIGN.md を生成する

以下の仕様に沿って `DESIGN.md` を生成する。

### YAML フロントマター

```yaml
---
version: alpha
name: <プロジェクト名>
description: <1行のプロジェクト説明>
colors:
  primary: "#XXXXXX"
  secondary: "#XXXXXX"
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
---
```

### トークンルール

- **Color**: `#` + 6桁の16進数（例: `"#1A1C1E"`）— 必ずダブルクォートで囲む
- **Dimension**: 数値 + 単位（`px`、`em`、`rem`）— 例: `48px`、`-0.02em`
- **トークン参照**: `{path.to.token}` — クォート不要
- `colors.primary` は必須（省略すると `missing-primary` 警告）
- コンポーネントの `backgroundColor` / `textColor` ペアは WCAG AA のコントラスト比（4.5:1）を確保する

### Markdown 本文（セクション順序は固定）

```markdown
## Overview

[ブランドパーソナリティ・ターゲットユーザー・UI が伝えるべき印象を 3〜5 文で記述]

## Colors

- **Primary (#XXXXXX):** [説明]
- **Secondary (#XXXXXX):** [説明]

## Typography

[タイポグラフィ戦略。使用フォントと各レベルの役割]

## Layout

[レイアウトとスペーシング戦略]

## Elevation & Depth

[視覚的階層の表現方法]

## Shapes

[ボーダーラジアスと形状の言語]

## Components

[主要コンポーネントのスタイルガイドライン]

## Do's and Don'ts

- Do [推奨プラクティス]
- Don't [禁止プラクティス]
```

---

## Step 4: リント検証

ファイル生成後にリンターを実行する（`<pm>` はプロジェクトで使用するパッケージマネージャーに置き換える）：

```bash
<pm> @google/design.md lint DESIGN.md
```

- **error** → `DESIGN.md` を修正して再実行する（エラーがなくなるまで繰り返す）
- **warning** → 確認して可能な限り修正する
- **info** → 対応不要

### よくあるリントエラーと修正方法

| エラー | 原因 | 修正 |
|---|---|---|
| `broken-ref` | トークン参照 `{colors.xxx}` が存在しない | トークン名を確認して修正する |
| `missing-primary` | `colors.primary` が未定義 | `colors` セクションに `primary` を追加する |
| `contrast-ratio` | テキスト/背景のコントラスト比が 4.5:1 未満 | コントラストの高い色に調整する |
| `section-order` | セクションが正しい順序になっていない | Overview → Colors → … の順に並べ直す |

---

## Step 5: 完了レポートを出力する

以下を報告する：
- 生成した `DESIGN.md` のパス
- リント結果（エラー・警告の件数）
- 主要デザイントークンの概要（カラー・タイポグラフィ）
