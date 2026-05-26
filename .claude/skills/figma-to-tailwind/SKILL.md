---
name: figma-to-tailwind
description: Figma ファイル URL からすべての変数を読み取り、Tailwind CSS v4 の @theme トークンとして src/index.css に書き出す。
argument-hint: <figma-url>
allowed-tools: mcp__plugin_figma_figma__use_figma, Read, Edit, Bash
---

# Figma 変数を Tailwind v4 トークンとしてインポート

`$ARGUMENTS` で指定された Figma ファイル URL からすべての変数を取得し、Tailwind CSS v4 の `@theme` カスタムプロパティとして `src/index.css` に書き出す。

**出力言語:** このスキルを呼び出したユーザーと同じ言語で応答する。

## ステップ 1: URL の解析

`$ARGUMENTS` から fileKey を抽出する：

- `figma.com/design/:fileKey/...` → `:fileKey` を取得
- `?node-id=` クエリパラメータは無視してよい（変数はファイル全体に属するため）

## ステップ 2: 全変数の取得

`use_figma` と Plugin API を使用して、未使用のものを含むファイル内のすべての変数を取得する：

```js
const collections = await figma.variables.getLocalVariableCollectionsAsync();
const variables = await figma.variables.getLocalVariablesAsync();

const result = collections.map(col => ({
  id: col.id,
  name: col.name,
  modes: col.modes,
  defaultModeId: col.defaultModeId,
  variables: col.variableIds.map(varId => {
    const v = variables.find(x => x.id === varId);
    if (!v) return null;
    return {
      id: v.id,
      name: v.name,
      type: v.resolvedType,
      valuesByMode: v.valuesByMode
    };
  }).filter(Boolean)
}));

return result;
```

> **注意:** `get_variable_defs` はノードに適用済みの変数しか返さない。未使用の変数を含むすべての変数を取得するには、必ず `use_figma` + Plugin API を使用すること。

## ステップ 3: 変数を CSS カスタムプロパティにマッピング

Figma 変数を Tailwind v4 CSS カスタムプロパティに変換する。

### 名前変換ルール

Figma の変数名（スラッシュ区切り）を kebab-case に変換する：
- コレクション名はコメントとして使用する
- 最後のパスセグメントがプロパティ名になる（例: `colors/brand/500` → `brand-500`）
- スラッシュ区切りのグループはハイフンで結合する（例: `font/size/xl` → `text-xl`）
- Figma が名前内の小数をハイフンで表している場合（`0-5`、`1-5`）、CSS ではアンダースコアに変換する（`0_5`、`1_5`）

### 型別プロパティ名プレフィックス

| Figma 型 | コレクション/変数名のヒント | CSS カスタムプロパティプレフィックス | 例 |
|---|---|---|---|
| `COLOR` | `colors/*` | `--color-` | `--color-brand-500` |
| `FLOAT` | `size/*` | `--spacing-` | `--spacing-4` |
| `FLOAT` | `radius/*` | `--radius-` | `--radius-md` |
| `FLOAT` | `border/*` | `--border-` | `--border-2` |
| `FLOAT` | `font/size/*` | `--text-` | `--text-base` |
| `FLOAT` | `font/weight/*` | `--font-weight-` | `--font-weight-bold` |
| `STRING` | `font/family/*` | `--font-` | `--font-base` |
| `STRING` | その他 | `--` | `--easing-default` |

### COLOR 値の変換

- Figma のカラーは 0〜1 の範囲で `{ r, g, b, a }` として返される
- `r, g, b` を 0〜255 の整数に変換し、2桁の16進数でフォーマット: `#rrggbb`
- alpha < 1.0 の場合は `rgba(r, g, b, a)` 形式を使用（r, g, b は 0〜255 の整数）
- 例: `{ r: 0.533, g: 0.414, b: 0.347, a: 1 }` → `Math.round(0.533*255) = 136 = 0x88` → `#886a59`

### FLOAT 値の変換

- px を rem に変換（÷ 16）
- 例外: 値 `0` → `0`; px 型変数名で値 `1` → `1px`

### モードの処理

- コレクションに複数のモード（ライト/ダークなど）がある場合は、**`defaultModeId`** モードの値を使用する

## ステップ 4: src/index.css への書き出し

### 既存ファイルの確認

`src/index.css` を読み込み、`@theme` ブロックが既に存在するか確認する。

### 書き出しパターン

**ケース A: `@theme` ブロックが既に存在する場合**

既存の `@theme` ブロック内に変数を追記する。キーが既に存在する場合は上書き（更新）する。

**ケース B: `@theme` ブロックが存在しない場合**

`@import "tailwindcss";` の直後に新しい `@theme` ブロックを挿入する：

```css
@import "tailwindcss";

@theme {
  /* Colors */
  --color-brand-500: #886a59;
}
```

### 出力フォーマット

コレクション/グループごとにコメントでセクションを区切る：

```css
@theme {
  /* Brand */
  --color-brand-50: #fff2ea;
  --color-brand-500: #886a59;

  /* Spacing (primitives/size) */
  --spacing-0: 0;
  --spacing-px: 1px;
  --spacing-4: 1rem;

  /* Border Radius (primitives/radius) */
  --radius-none: 0;
  --radius-md: 0.375rem;
  --radius-full: 9999px;

  /* Font Size (primitives/font/size) */
  --text-base: 0.9375rem;
  --text-xl: 1.25rem;

  /* Font Family (primitives/font/family) */
  --font-base: "Noto Sans JP", sans-serif;

  /* Font Weight (primitives/font/weight) */
  --font-weight-bold: 700;
}
```

## ステップ 5: 完了報告

以下を報告する：
- インポートしたコレクション数と変数の合計数
- 追加・更新した CSS カスタムプロパティの一覧（グループ別の件数）
- 使用したモード名（複数モードがある場合）
- `src/index.css` のパス

## エラーハンドリング

- URL から fileKey を抽出できない場合: 有効な Figma デザインファイル URL を提供するようユーザーに確認する
- 変数が 0 件の場合: ファイルに変数が定義されていないことを報告する
- `src/index.css` が見つからない場合: 現在のディレクトリにファイルが存在しないことを報告し、出力パスをユーザーに確認する
- `use_figma` がエラーを返した場合: リトライ前にエラーメッセージを確認する（操作はアトミックのため、部分的な書き込みは発生しない）
