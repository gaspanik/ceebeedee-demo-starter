---
description: Figma ファイルの Variables をすべて読み取り、Tailwind CSS v4 の @theme トークンとして src/index.css に書き出す。Figma の design ファイル URL を渡して使う。
tools:
  - mcp_com_figma_mcp_use_figma
  - search/codebase
  - edit/editFiles
---

# Import Figma Variables as Tailwind v4 Tokens

ユーザーが提供した Figma ファイル URL からすべての Variables を取得し、Tailwind CSS v4 の `@theme` カスタムプロパティとして `src/index.css` に書き出す。

**回答言語:** ユーザーが使用している言語で回答すること。

## Step 1: URL を解析する

ユーザーが提供した Figma URL から fileKey を取得する:

- `figma.com/design/:fileKey/...` → `:fileKey` を使用
- `?node-id=` クエリパラメータは無視してよい（Variables はファイル全体に属する）

## Step 2: 全 Variables を取得する

`use_figma` で Plugin API を使い、ファイル内のすべての Variables（未使用のものを含む）を取得する:

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

> **注意:** `get_variable_defs` はノードに適用済みの Variables しか返さない。未使用を含む全 Variables を取得するには必ず `use_figma` + Plugin API を使うこと。

### Variable Alias の処理

`valuesByMode` の値が `{ type: "VARIABLE_ALIAS", id: "..." }` の形式の場合、参照先の変数を `variables` 配列から `id` で検索し、その `valuesByMode` から `defaultModeId` の値を使用する。循環参照が発生した場合（参照チェーンが自分自身に戻る）はその変数をスキップし、報告に含める。

## Step 3: CSS カスタムプロパティにマッピングする

Figma Variables を Tailwind v4 の CSS カスタムプロパティに変換する。

### 名前変換ルール

Figma の変数名（スラッシュ区切り）をケバブケースに変換する:
- コレクション名はそのままの文字列でコメントに使用する（例: `/* Brand */`）。コレクションごとに1つのコメントを出力する。
- コレクション名部分（パスの先頭セグメント）を除いた残りのパスセグメントを、すべてハイフンで結合してプロパティ名とする（例: `colors/brand/500` → コレクション `colors` を除いて `brand-500`）
- 型別プレフィックステーブルで決まる CSS プレフィックスを先頭に付与する（例: `font/size/xl` → コレクション `font` 配下の `size/xl` → プレフィックス `--text-` + `size-xl` → `--text-size-xl`）
- Figma の変数名に数値の小数点がハイフンとして含まれる場合（前後が数字のハイフン、例: `0-5` は `0.5` を意味する）、CSS カスタムプロパティ名ではアンダースコアに変換する（例: `--spacing-0_5`）

### 型別プロパティ名プレフィックス

パターンマッチは**テーブルの上から順に**最初に一致したものを使用する。より具体的なパス（例: `font/size/*`）が先に記載されているため、上位ルールが優先される。

| Figma 型 | コレクション/変数名のヒント | CSS カスタムプロパティプレフィックス | 例 |
|---|---|---|---|
| `COLOR` | `colors/*` | `--color-` | `--color-brand-500` |
| `FLOAT` | `font/size/*` | `--text-` | `--text-base` |
| `FLOAT` | `font/weight/*` | `--font-weight-` | `--font-weight-bold` |
| `FLOAT` | `size/*` | `--spacing-` | `--spacing-4` |
| `FLOAT` | `radius/*` | `--radius-` | `--radius-md` |
| `FLOAT` | `border/*` | `--border-` | `--border-2` |
| `STRING` | `font/family/*` | `--font-` | `--font-base` |
| `STRING` | その他 | `--` | `--easing-default`（変数のフルパスをスラッシュ→ハイフン変換: `easing/default` → `--easing-default`） |

### COLOR 値の変換

- Figma の色は `{ r, g, b, a }` で 0〜1 の範囲で返される
- `r, g, b` を 0〜255 の整数に変換し、2桁の16進数でフォーマット: `#rrggbb`
- alpha < 1.0 の場合は `rgba(r, g, b, a)` 形式（r, g, b は 0〜255 整数）
- 例: `{ r: 0.533, g: 0.414, b: 0.347, a: 1 }` → `Math.round(0.533*255) = 136 = 0x88` → `#886a59`

### FLOAT 値の変換

- px を rem に変換（÷ 16）
- 例外: 値 `0` → `0`；値 `1` かつ変数名に `px` または `border` が含まれる場合 → `1px`

### モード対応

- コレクションに複数モード（ライト/ダークなど）がある場合は **`defaultModeId`** のモードの値を使用

## Step 4: src/index.css に書き出す

### 既存ファイルの確認

`src/index.css` を読み込み、`@theme` ブロックが既に存在するか確認する。

### 書き込みパターン

**ケース A: `@theme` ブロックが既に存在する**

既存の `@theme` ブロック内に変数を追記する。同一の CSS カスタムプロパティ名（例: `--color-brand-500`）が既に存在する場合、その行の値部分のみを新しい値に置き換える。コメント行は変更しない。

**ケース B: `@theme` ブロックは存在しないが `@import "tailwindcss";` は存在する**

`@import "tailwindcss";` の直後に新しい `@theme` ブロックを挿入する:

```css
@import "tailwindcss";

@theme {
  /* Colors */
  --color-brand-500: #886a59;
}
```

**ケース C: `@import "tailwindcss";` も存在しない**

ファイルの先頭に `@import "tailwindcss";` と `@theme` ブロックを追加する:

```css
@import "tailwindcss";

@theme {
  /* Colors */
  --color-brand-500: #886a59;
}
```

### 出力フォーマット

コレクション/グループごとにコメントで区切る:

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

## Step 5: 完了を報告する

以下を報告する:
- インポートしたコレクション数と合計 Variables 数
- 追加/更新した CSS カスタムプロパティの一覧（グループ別件数）
- 使用したモード名（複数モードがある場合）
- `src/index.css` のパス

## エラーハンドリング

- URL から fileKey を取得できない場合: 有効な Figma design ファイル URL を提供するようユーザーに確認する
- Variables が 0 件の場合: ファイルに Variables が定義されていないことを報告する
- `src/index.css` が見つからない場合: ファイルが現在のディレクトリに存在しないことを報告し、出力パスを確認する
- `use_figma` がエラーを返した場合: 再試行前にエラーメッセージを確認する（操作はアトミック — 部分書き込みは発生しない）
