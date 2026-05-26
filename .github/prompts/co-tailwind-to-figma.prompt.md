---
description: src/index.css の Tailwind CSS v4 @theme トークンを読み取り、Figma Variables として書き出す。引数は「Figma URL（既存ファイルに追記）」「ファイル名（新規作成）」「省略（"Design Tokens" という名前で新規作成）」の3パターン。
tools:
  - mcp_com_figma_mcp_use_figma
  - mcp_com_figma_mcp_create_new_file
  - mcp_com_figma_mcp_whoami
  - search/codebase
  - edit/editFiles
---

# Export Tailwind v4 Tokens as Figma Variables

`src/index.css` の `@theme` ブロックにある CSS カスタムプロパティをすべて読み取り、Figma Variables として書き出す。

**回答言語:** ユーザーが使用している言語で回答すること。

## Step 1: src/index.css を読み取り解析する

`src/index.css` を読み込み、`@theme { ... }` ブロック内のすべての CSS カスタムプロパティを抽出する。

### 変数プレフィックスとその分類

分類は以下の**優先順位**で適用する（上位ルールが一致した時点で決定し、以降は評価しない）:

1. **完全一致の特殊名を最優先**: `--default-font-family`、`--heading-font-family` → Typography / STRING
2. **最も具体的なプレフィックスを次に優先**: `--font-weight-*`、`--font-size-*`、`--text-*`（値が数値/rem/px）→ Typography / FLOAT
3. **汎用プレフィックス**: `--font-*`（値がフォントファミリー文字列、つまりクォートを含むかフォント名らしい文字列）→ Typography / STRING。値が数値/rem/px の場合は FLOAT ルール（上記 2）を適用する
4. **その他のプレフィックス**: 下表に従う
5. **フォールバック**: どれにも一致しない場合は Other コレクションへ

| CSS 変数プレフィックス | カテゴリ | Figma コレクション | Figma 変数型 |
|---|---|---|---|
| `--color-*` | Color | `Colors` | `COLOR` |
| `--*-color-*` | Color | `Colors` | `COLOR` |
| `--*-font-family` / `--font-family-*` / `--default-font-family` / `--heading-font-family` | Font family | `Typography` | `STRING` |
| `--font-*`（値がフォントファミリー文字列） | Font family | `Typography` | `STRING` |
| `--font-weight-*` | Font weight | `Typography` | `FLOAT` |
| `--font-size-*` | Font size | `Typography` | `FLOAT` |
| `--text-*`（値が数値/rem/px） | Font size | `Typography` | `FLOAT` |
| `--spacing-*` | Spacing | `Spacing` | `FLOAT` |
| `--radius-*` | Border radius | `Radius` | `FLOAT` |
| `--border-*`（数値） | Border width | `Border` | `FLOAT` |
| その他 `--*` | Other | `Other` | 値の内容で判定（下記参照） |

**Other コレクションの型判定ルール:**
- 値がカラーパターン（`#rrggbb`、`rgb()`、`rgba()`、`oklch()`、`hsl()` など）→ `COLOR`
- 値が数値/rem/px → `FLOAT`
- 値がクォートされた文字列またはフォント名らしい文字列 → `STRING`
- それ以外（`var(--x)`、`linear-gradient()` など複雑な値）→ スキップして警告を報告

### 名前変換ルール（CSS → Figma スラッシュ記法）

Figma の変数名はスラッシュ区切りのパス形式を使う:

- `--color-primary` → `primary`（Colors コレクション内）
- `--color-brand-500` → `brand/500`
- `--default-font-family` → `default`（Typography コレクション内）
- `--heading-font-family` → `heading`
- `--font-sans` → `sans`
- `--text-base` → `size/base`（Typography コレクション内）
- `--spacing-4` → `4`（Spacing コレクション内）
- `--radius-md` → `md`（Radius コレクション内）

プレフィックス（`--color-`、`--spacing-` など）を除いた残りを変数名として使用する。

## Step 2: CSS 値を Figma 値に変換する

### COLOR 変換

- `#rrggbb` → `{ r: parseInt(rr,16)/255, g: parseInt(gg,16)/255, b: parseInt(bb,16)/255, a: 1 }`
- `#rrggbbaa` → alpha を含めて変換
- `rgba(r, g, b, a)` → `{ r: r/255, g: g/255, b: b/255, a: a }`
- `rgb(r, g, b)` → `{ r: r/255, g: g/255, b: b/255, a: 1 }`
- `oklch(L C H)` → 標準の oklch-to-sRGB 行列を使って sRGB に変換してから Figma カラーオブジェクトを作成する
- `hsl(H S% L%)` → RGB に変換してから Figma カラーオブジェクトを作成する
- `color-mix()` やその他の複雑なカラー関数 → スキップして警告を報告する

### STRING 変換（フォントファミリー）

- `"Font Name", fallback` → `"Font Name"`（最初のクォートされたフォント名のみ抽出）
- `FontName, fallback` → `"FontName"`
- クォートと最初のカンマ以降（フォールバックフォント）を除去する

### FLOAT 変換（数値）

- `1rem` → `16`（px 換算で × 16）
- `0.5rem` → `8`
- `16px` → `16`（px を除去）
- `0` → `0`
- `1px` → `1`
- 純粋な数値はそのまま通す

## Step 3: Figma ファイルを準備する

ユーザーの入力に応じて分岐する:

### パターン A: Figma URL が提供された場合（既存ファイルに追記）

URL から `fileKey` を取得する。`whoami` や `create_new_file` の呼び出しは不要。

- `figma.com/design/:fileKey/...` → `:fileKey` を使用
- `?node-id=` クエリパラメータは無視してよい

以降の `use_figma` 呼び出しには取得した `fileKey` を使用する。

### パターン B: ファイル名が提供された場合、または引数なしの場合（新規作成）

まず `whoami` を呼び出してプラン一覧を取得する:

- **1プラン** → その `key` を `planKey` として使用
- **複数プラン かつ ファイル名が指定されていない場合** → どのチーム/組織にファイルを作成するかユーザーに確認してから進む
- **複数プラン かつ ファイル名が指定されている場合** → 最初のプランの `key` を `planKey` として使用し、確認なしで進む

`create_new_file` で新規ファイルを作成する:

```
create_new_file({
  fileName: "<引数のファイル名、または "Design Tokens">",
  planKey: "<whoami で取得した planKey>",
  editorType: "design"
})
```

返された `fileKey` を以降の `use_figma` 呼び出しに使用する。

## Step 4: Figma Variables を作成する

`use_figma` と Plugin API を使ってコレクションと Variables を一括作成する。

**変数が 0 件のコレクションはスキップする** — 少なくとも 1 つのトークンがあるカテゴリのみコレクションを作成する。

以下のコードを `use_figma` に渡す（`tokenData` には前のステップで抽出した実際の値を入れること）:

```javascript
// ===== 実際のトークンデータ（Steps 1–2 で抽出した値を入れる） =====
const tokenData = {
  Colors: [
    // 例: { name: "primary", type: "COLOR", value: { r: 0.173, g: 0.094, b: 0.063, a: 1 } }
  ],
  Typography: [
    // 例: { name: "default", type: "STRING", value: "Gen Interface JP" }
    // 例: { name: "size/base", type: "FLOAT", value: 16 }
  ],
  Spacing: [
    // 例: { name: "4", type: "FLOAT", value: 16 }
  ],
  Radius: [],
  Border: [],
  Other: [],
};
// ================================================================

const results = [];

for (const [collectionName, variables] of Object.entries(tokenData)) {
  if (variables.length === 0) continue;

  const collections = await figma.variables.getLocalVariableCollectionsAsync();
  let collection = collections.find(c => c.name === collectionName);
  if (!collection) {
    collection = figma.variables.createVariableCollection(collectionName);
  }
  const modeId = collection.defaultModeId;

  for (const token of variables) {
    const existingVars = await figma.variables.getLocalVariablesAsync();
    let variable = existingVars.find(
      v => v.variableCollectionId === collection.id && v.name === token.name
    );
    const isNew = !variable;

    if (!variable) {
      variable = figma.variables.createVariable(token.name, collection, token.type);
    }


    variable.setValueForMode(modeId, token.value);
    results.push({ path: `${collectionName}/${token.name}`, type: token.type, action: isNew ? 'created' : 'updated' });
  }
}

return {
  total: results.length,
  created: results.filter(r => r.action === 'created').length,
  updated: results.filter(r => r.action === 'updated').length,
  variables: results
};
```

### use_figma 呼び出し時の注意

- 実行前に `tokenData` には必ず前のステップで抽出した **実際の値** を入れること
- **同名の変数が既に存在する場合は `setValueForMode` で値を上書き更新する。スキップしない。**
- 合計変数数が 30 以下の場合: すべてのコレクションを 1 回の `use_figma` 呼び出しで処理する
- 合計変数数が 31 以上の場合: コレクションごとに 1 回ずつ `use_figma` を呼び出し、順番に処理する

## Step 5: 完了を報告する

以下を報告する:

- Figma ファイル名と fileKey（または URL）
- 作成したコレクションの一覧とコレクションごとの変数件数
- 新規作成した変数の件数と更新した変数の件数（コレクション別）
- スキップしたコレクション（空だったもの）
- 変換できなかった変数とその理由

## エラーハンドリング

- `src/index.css` が見つからない場合: ファイルパスをユーザーに確認する
- `@theme` ブロックが空の場合: 変数が定義されていないことを報告する
- `create_new_file` が失敗した場合: Figma のログイン状態を確認するようユーザーに伝える（`whoami` で確認可能）
- `use_figma` がエラーを返した場合: エラー詳細を確認し、変数データのフォーマットを見直す
- 変換できない値（`var(--other)` 参照、`linear-gradient` など）は警告としてスキップし、報告に含める
