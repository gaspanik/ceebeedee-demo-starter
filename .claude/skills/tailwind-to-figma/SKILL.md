---
name: tailwind-to-figma
description: src/index.css の Tailwind CSS v4 @theme トークンを読み取り、型別に適切なコレクションへ整理して Figma ファイルに変数として書き出す。
argument-hint: (省略可: Figma ファイル URL → 既存ファイルに追記 / ファイル名のみ → その名前で新規作成 / 省略 → "Design Tokens" という名前で新規作成)
allowed-tools: mcp__plugin_figma_figma__use_figma, mcp__plugin_figma_figma__create_new_file, mcp__plugin_figma_figma__whoami, Read, Bash, Skill
---

# Tailwind v4 トークンを Figma 変数としてエクスポート

`src/index.css` の `@theme` ブロックからすべての CSS カスタムプロパティを読み取り、Figma 変数として書き出す。

**出力言語:** このスキルを呼び出したユーザーと同じ言語で応答する。

## ステップ 0: figma-use スキルの読み込み

`use_figma` を呼び出す前に、必ず最初に `figma-use` スキルを読み込む：

```
/figma:figma-use
```

## ステップ 1: src/index.css の読み取りと解析

`Read` ツールで `src/index.css` を読み込み、`@theme { ... }` ブロック内のすべての CSS カスタムプロパティを抽出する。

### 変数プレフィックスと分類

| CSS 変数プレフィックス | カテゴリ | Figma コレクション | Figma 変数型 |
|---|---|---|---|
| `--color-*` | カラー | `Colors` | `COLOR` |
| `--*-color-*` | カラー | `Colors` | `COLOR` |
| `--*-font-family` / `--font-family-*` / `--default-font-family` / `--heading-font-family` | フォントファミリー | `Typography` | `STRING` |
| `--font-*`（文字列値） | フォントファミリー | `Typography` | `STRING` |
| `--text-*`（数値/rem/px） | フォントサイズ | `Typography` | `FLOAT` |
| `--font-size-*` | フォントサイズ | `Typography` | `FLOAT` |
| `--font-weight-*` | フォントウェイト | `Typography` | `FLOAT` |
| `--spacing-*` | スペーシング | `Spacing` | `FLOAT` |
| `--radius-*` | 角丸 | `Radius` | `FLOAT` |
| `--border-*`（数値） | ボーダー幅 | `Border` | `FLOAT` |
| その他 `--*` | その他 | `Other` | 値によって決定 |

### 名前変換ルール（CSS → Figma スラッシュ記法）

Figma の変数名はスラッシュ区切りのパス形式を使用する：

- `--color-primary` → `primary`（Colors コレクション内）
- `--color-brand-500` → `brand/500`
- `--default-font-family` → `default`（Typography コレクション内）
- `--heading-font-family` → `heading`
- `--font-sans` → `sans`
- `--text-base` → `size/base`（Typography コレクション内）
- `--spacing-4` → `4`（Spacing コレクション内）
- `--radius-md` → `md`（Radius コレクション内）

プレフィックス（`--color-`、`--spacing-` など）を除去し、残りを変数名として使用する。

## ステップ 2: CSS 値を Figma 値に変換

### COLOR 変換

- `#rrggbb` → `{ r: parseInt(rr,16)/255, g: parseInt(gg,16)/255, b: parseInt(bb,16)/255, a: 1 }`
- `#rrggbbaa` → アルファ付きで変換
- `rgba(r, g, b, a)` → `{ r: r/255, g: g/255, b: b/255, a: a }`
- `rgb(r, g, b)` → `{ r: r/255, g: g/255, b: b/255, a: 1 }`

### STRING 変換（フォントファミリー）

- `"Font Name", fallback` → `"Font Name"`（最初の引用符付きフォント名のみ抽出）
- `FontName, fallback` → `"FontName"`
- 引用符と最初のカンマ以降（フォールバックフォント）を除去する

### FLOAT 変換（数値）

- `1rem` → `16`（px 換算のため × 16）
- `0.5rem` → `8`
- `16px` → `16`（px を除去）
- `0` → `0`
- `1px` → `1`
- 単純な数値はそのままスルー

## ステップ 3: Figma ファイルの準備

`$ARGUMENTS` に基づいて分岐する：

### パターン A: Figma URL が指定された場合（既存ファイルに追記）

URL から `fileKey` を抽出する。`whoami` や `create_new_file` を呼び出す必要はない。

- `figma.com/design/:fileKey/...` → `:fileKey` を取得
- `?node-id=` クエリパラメータは無視してよい

抽出した `fileKey` を後続の `use_figma` 呼び出しで使用する。

### パターン B: ファイル名が指定された場合、または引数なしの場合（新規ファイルを作成）

最初に `whoami` を呼び出してプランの一覧を取得する：

- **1 プラン** → その `key` を `planKey` として使用する
- **複数のプラン** → 処理を進める前に、どのチーム/組織にファイルを作成するかをユーザーに確認する

`create_new_file` で新規ファイルを作成する：

```
create_new_file({
  fileName: "<引数の名前または "Design Tokens">",
  planKey: "<whoami から取得した planKey>",
  editorType: "design"
})
```

返された `fileKey` を後続の `use_figma` 呼び出しで使用する。

## ステップ 4: Figma 変数の作成

`use_figma` と Plugin API を使用して、コレクションと変数を一括で作成する。

**変数が存在しないコレクションはスキップする** — 少なくとも1つのトークンがあるカテゴリのみコレクションを作成する。

以下のコードを `use_figma` に渡す（`tokenData` には前のステップで抽出した実際の値を入力すること）：

```javascript
// ===== 実際のトークンデータ（ステップ 1〜2 で抽出した値を入力） =====
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
// =====================================================================

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

    if (!variable) {
      variable = figma.variables.createVariable(token.name, collection, token.type);
    }

    variable.setValueForMode(modeId, token.value);
    results.push(`${collectionName}/${token.name} (${token.type})`);
  }
}

return { created: results.length, variables: results };
```

### use_figma 呼び出し時の注意事項

- 実行前に必ず `tokenData` に前のステップで抽出した**実際の値**を入力すること
- 状況に応じて、すべてのコレクションを1回の呼び出しで処理するか、コレクションごとに分割するかを判断する
- 変数の数が多い場合（30件以上）は、コレクションごとに呼び出しを分割する

## ステップ 5: 完了報告

以下を報告する：

- Figma ファイル名と fileKey（または URL）
- 作成したコレクションの一覧とコレクション別の変数数
- 作成した変数の一覧（コレクション別）
- スキップしたコレクション（空だったもの）
- 変換できなかった変数とその理由

## エラーハンドリング

- `src/index.css` が見つからない場合: ファイルパスをユーザーに確認する
- `@theme` ブロックが空の場合: 変数が定義されていないことを報告する
- `create_new_file` が失敗した場合: Figma のログイン状態を確認するようユーザーに案内する（`whoami` で確認可能）
- `use_figma` がエラーを返した場合: エラーの詳細を確認し、変数データのフォーマットを見直す
- 変換できない値（例: `var(--other)` 参照、`linear-gradient` など）は警告付きでスキップし、報告に含める
