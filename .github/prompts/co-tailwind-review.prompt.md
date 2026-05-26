---
description: Tailwind CSS コードのレビュー・最適化・移行、および HTML アクセシビリティチェックを行う。v3→v4 移行、任意値の整理、冗長クラスの検出、HTML の a11y 確認が必要な場合に使う
tools:
  - search/codebase
  - edit/editFiles
  - execute/getTerminalOutput
  - execute/runInTerminal
---

Tailwind CSS コードを 5 つの観点でレビューし、改善提案または自動修正を行います。

**返答言語:** 日本語で返答する。

---

## 開始メッセージ

レビューを始める前に、必ず以下のメッセージをユーザーに表示する：

```
> **注意:** 自動修正を適用する場合は、変更を元に戻せる状態にしておいてください。
> Git を使用している場合は `git status` で未コミットの変更を確認してください。
> 未保存の作業がある場合は、`git stash` または `git commit` を先に実行することを推奨します。
```

このメッセージを表示したら、ステップ 0 のコマンドをすぐに実行する（ユーザーの返答は待たない）。ただし判断が必要な箇所（バージョン選択・修正承認など）では必ずユーザーに確認を求める。

---

## ステップ 0: デザインシステム定義の検出

レビューを始める前に、プロジェクトルートに **`DESIGN.md`** が存在するかを確認する。

```bash
ls DESIGN.md 2>/dev/null || ls docs/DESIGN.md 2>/dev/null || ls .design/DESIGN.md 2>/dev/null
```

**見つかった場合:**
- ファイルを全文読み込む
- 定義されているトークンを抽出する：カラー名・値、フォントファミリー、スペーシングスケール、タイプスケール
- ディメンション 3 の **優先度 1** 参照として保存する
- ユーザーに簡潔な通知を表示する：

```
DESIGN.md が見つかりました — そこで定義されているデザイントークンを提案の主要参照として使用します。
```

**見つからなかった場合:** そのまま続行する。`@theme` 変数と Tailwind 標準スケールがフォールバックとなる。

---

## ステップ 0.5: Tailwind バージョンの検出

```bash
# 1. package.json でバージョンを確認
cat package.json | grep '"tailwindcss"'

# 2. tailwind.config.js の有無を確認（v3 の指標）
ls tailwind.config.js 2>/dev/null && echo "found"

# 3. CSS エントリーファイルのインポートスタイルを確認
grep -r "@tailwind\|@import.*tailwindcss" src/ --include="*.css" | head -5
```

**判定基準:**

> **優先度:** `package.json` のバージョン番号を最優先とする。`package.json` に記載がない場合のみ、CSS インポートスタイルで判定する。`tailwind.config.js` の有無は補助情報として使用し、単独では判定に使わない。

| 検出結果 | 判定 |
|---|---|
| `"tailwindcss": "^3.*"` または `"tailwindcss": "3.*"` | **v3** |
| `"tailwindcss": "^4.*"` または `"tailwindcss": "4.*"` | **v4** |
| `tailwind.config.js` が存在する | おそらく **v3** |
| CSS に `@tailwind base;` が含まれる | **v3** |
| CSS に `@import "tailwindcss";` が含まれる | **v4** |

**v3 が検出された場合 → ユーザーに選択肢を提示する:**

```
Tailwind CSS v3 が検出されました。

どのように進めますか？

  1. レビューのみ — 問題を報告するが書き換えはしない
  2. v4 へ移行 — v4 構文に書き直す

どちらをご希望ですか？ (1 / 2)
```

- **選択肢 1（レビューのみ）:** ディメンション 2 は v3 コードの問題のみを報告。書き換えなし。
- **選択肢 2（移行）:** ディメンション 2 の変換テーブルに基づき、自動変換できるものを一括で適用する。手動判断が必要な変換（`space-y-*` の文脈依存パターンなど）は変換リストに含めず、別途提案として報告する。

**v4 が検出された場合:**
ディメンション 2 で v3 のパターンが混入していないかを確認する。

**不明の場合:**
ユーザーに確認する：`Tailwind のバージョンを判定できませんでした。v3 と v4 のどちらを使用していますか？`

---

## ステップ 1: 対象ファイルの特定

ファイルが指定されていない場合、プロジェクト内の HTML / JSX / TSX / Vue / Svelte / Astro ファイルで Tailwind クラスを含むものを検索する。

```bash
grep -rl "class=" . --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.astro" | grep -v "node_modules" | grep -v "dist"
```

### ステップ 1.5: レビュー範囲の確認（HTML ファイルのみ）

`.html` および `.astro` ファイルに適用する。`.tsx` / `.jsx` / `.vue` などはスキップする。

ファイルを読み込んでサイズを判定する：
- 以下のいずれかに該当する場合に範囲確認を行う：(a) ファイルが 80 行以上、または (b) `<section>` / `id` 付き `<div>` が 3 つ以上存在する。どちらか一方でも該当すれば確認を行う
- どちらにも該当しない場合 → ファイル全体をレビューする

**範囲確認が必要な場合、セクションを自動検出して提示する:**

以下の優先順位でセクション境界を検出する：
1. `<section>` タグ（`id` または `aria-label` の値を使用）
2. トップレベルの `<div id="...">` ブロック
3. `<!-- ... -->` コメントで区切られたブロック
4. セマンティック要素：`<header>` / `<main>` / `<footer>` / `<nav>`

セクションリストを表示し、以下の選択肢でユーザーに確認する：

```
1. 全体（推奨）  — ページ全体をレビュー
2. 前半         — 上部のセクション
3. 後半         — 下部のセクション
4. 番号で指定   — セクション番号を入力
```

---

## ステップ 2: 5 つのディメンションでレビューする

**すべての** ディメンションを確認する。1 件でも見つかれば報告する。**各ディメンションの結果を個別のセクションに蓄積しながら順番に処理する。次のディメンションに進む前に現在のディメンションの発見事項をすべて記録すること。**

---

## ディメンション 1: クラスの冗長性チェック

同一要素上の **競合クラス** または **無意味な重複** を検出する。

**よくあるパターン:**
- レイアウトの競合：`block` と `flex` が共存、`hidden` と `flex` が共存
- 方向の競合：`flex-row` と `flex-col` が共存
- サイズの重複：`w-full` と `w-1/2` が共存

**報告フォーマット:**
```
[冗長性] <ファイル名>:<行>
  問題: `flex` と `block` の両方が指定されている
  現在: class="flex block px-4"
  修正:  class="flex px-4"  # `block` を削除 — `flex` が display を設定する
```

### 共有クラスの検出（`*:` バリアントの提案）

親要素の**すべて（またはほとんど）の直接子要素が同じクラスを共有している**場合、
`*:` バリアントを使って親にまとめることを提案する。

**検出条件:**
- 同じ親を持つ兄弟要素が 3 つ以上
- 兄弟要素の 75% 以上（最低 3 つ）が **2 つ以上**の共通クラスを持つ

**報告フォーマット:**
```
[共有クラス] <ファイル名>:<行>
  問題: 3 つの <a> 要素がすべて "text-gray-800 hover:text-blue-600" を持つ
  現在:
    <ul>
      <li><a class="text-gray-800 hover:text-blue-600">Home</a></li>
      <li><a class="text-gray-800 hover:text-blue-600">About</a></li>
      <li><a class="text-gray-800 hover:text-blue-600">Contact</a></li>
    </ul>
  提案:
    <ul class="*:text-gray-800 *:hover:text-blue-600">
      <li><a>Home</a></li>
      <li><a>About</a></li>
      <li><a>Contact</a></li>
    </ul>
```

### cn() / クラスマージ関数の提案（React / TSX / Astro）

`.tsx` / `.jsx` / `.astro` ファイルに対しては、クラスマージ関数が使われているかも確認する。

```bash
grep -E '"clsx"|"tailwind-merge"|"class-variance-authority"' package.json
grep -r "cn(" --include="*.tsx" --include="*.ts" --include="*.astro" src/ | grep -v "node_modules" | head -5
```

未導入で冗長性の問題が 1 件以上見つかった場合、`clsx` + `tailwind-merge` の導入を提案する。

---

## ディメンション 2: v3 → v4 移行チェック

### カテゴリ別の変更点

**[削除] スペーシング / 分割**
| v3（削除済み） | v4（推奨） |
|---|---|
| `space-x-*` | `flex` + `gap-*` |
| `space-y-*` | `flex flex-col` + `gap-*` |
| `divide-x` / `divide-y` | 個々の子要素にボーダーを付ける |

**[削除] 不透明度ユーティリティ → スラッシュ構文**
| v3（削除済み） | v4（推奨） |
|---|---|
| `bg-opacity-50` | `bg-black/50` |
| `text-opacity-50` | `text-black/50` |
| `border-opacity-50` | `border-black/50` |

**[名称変更] Flex ユーティリティ**
| v3（旧） | v4（新） |
|---|---|
| `flex-shrink` / `flex-shrink-0` | `shrink` / `shrink-0` |
| `flex-grow` / `flex-grow-0` | `grow` / `grow-0` |

**[名称変更] スケールシフト（⚠️ 見た目がサイレントに変わる）**

| v3 | v4 | 実際のサイズ |
|---|---|---|
| `shadow-sm` | `shadow-xs` | 小さいシャドウ |
| `shadow`（サフィックスなし） | `shadow-sm` | デフォルトシャドウ |
| `blur-sm` | `blur-xs` | 小さいぼかし |
| `blur`（サフィックスなし） | `blur-sm` | デフォルトぼかし |
| `rounded-sm` | `rounded-xs` | 小さい角丸 |
| `rounded`（サフィックスなし） | `rounded-sm` | デフォルト角丸 |

**[変更] その他**
| v3 | v4 |
|---|---|
| `outline-none` | `outline-hidden` |
| `!flex` / `!bg-red-500` | `flex!` / `bg-red-500!` |
| `bg-[--brand-color]` | `bg-(--brand-color)` |
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` | `box-decoration-slice` |

### 検出

```bash
grep -rn \
  -e "space-x-" -e "space-y-" \
  -e "flex-shrink" -e "flex-grow" \
  -e "bg-opacity-" -e "text-opacity-" -e "border-opacity-" \
  -e "ring-opacity-" -e "divide-opacity-" -e "placeholder-opacity-" \
  -e "overflow-ellipsis" -e "outline-none" \
  --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.astro" \
  . | grep -v "node_modules" | grep -v "dist"
```

**報告フォーマット:**
```
[v4 移行] <ファイル名>:<行>
  問題: `shadow-sm` は v4 で `shadow-xs` に名称変更（スケールシフト）
  現在: class="shadow-sm rounded px-4"
  修正:  class="shadow-xs rounded-sm px-4"
```

### 移行の実行（v4 移行を選んだ場合のみ）

すべての変換リストをユーザーへ確認してから一括で適用する：

```
以下の変換を適用します（計 X 件）：

  [1] 12 行目  space-y-4        → flex flex-col gap-4
  [2] 18 行目  flex-shrink-0    → shrink-0
  ...

進めますか？（yes / 番号で除外例: "3, 4 をスキップ"）
```

**手動判断が必要な変換（自動適用しない）:**
- `space-y-*` / `space-x-*` → 親に `flex` があるか文脈に依存
- `shadow`（サフィックスなし） → デザインレビューが必要
- `!important` 修飾子 → 大量にある場合は慎重に

CSS エントリーファイルの `@tailwind base/components/utilities` → `@import "tailwindcss"` も提案する。

---

## ディメンション 3: デザイントークンの使用チェック

**任意値**を検出し、`@theme` 変数または Tailwind 標準スケール値への置き換えを提案する。

### 参照の優先順位

| 優先度 | 参照元 |
|---|---|
| **1** | `DESIGN.md`（ステップ 0 で読み込み済み） |
| **2** | CSS エントリーファイルの `@theme` |
| **3** | Tailwind 標準スケール |
| **4** | 任意値のまま維持（デザインシステムへの追加を提案） |

**報告フォーマット:**
```
[トークン] <ファイル名>:<行>
  問題: ハードコードされた色 `[#294779]`
  現在: class="text-[#294779]"
  提案:  class="text-primary"  # style.css に --color-primary: #294779 が定義されている
```

### 3-A: フォントファミリーの任意値

```bash
grep -rn "font-\[" --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.astro" . | grep -v "node_modules" | grep -v "dist"
```

`@theme` に `--font-*` 変数がなければ定義を提案する。

### 3-B: ピクセル / 単位スケールのマッピング

**フォントサイズ:** `text-[12px]`→`text-xs`, `text-[14px]`→`text-sm`, `text-[16px]`→`text-base`, `text-[18px]`→`text-lg`, `text-[20px]`→`text-xl`, `text-[24px]`→`text-2xl`, etc.

**スペーシング（1 単位 = 4px）:** px ÷ 4 = スケール番号（例: `w-[320px]` → `w-80`）

**フォントウェイト:** `font-[400]`→`font-normal`, `font-[500]`→`font-medium`, `font-[600]`→`font-semibold`, `font-[700]`→`font-bold`

### 3-C: Tailwind 組み込みカラーのマッチング

hex 値を Tailwind カラーパレットと照合する。完全一致の場合のみ自動提案する。一致しない場合は最も近い Tailwind カラーを参考として括弧内に表示し、目視での確認を必須とする。

```
[トークン] <ファイル名>:<行>
  問題: ハードコードされた色 `[#374151]` は Tailwind 組み込みカラーと一致
  現在: class="text-[#374151]"
  修正:  class="text-gray-700"
```

### 3-D: 簡略化できる任意値（v4）

| パターン | 例 | 修正 |
|---|---|---|
| `aspect-[N/M]` | `aspect-[16/7]` | `aspect-16/7` |
| `w-[N/M]` | `w-[1/2]` | `w-1/2` |

---

## ディメンション 4: アクセシビリティチェック（Tailwind）

**大文字テキスト:**
HTML に直接大文字で書かれているテキストを検出する。`uppercase` クラスで視覚的に大文字にする。

```
[a11y] <ファイル名>:<行>
  問題: テキストが HTML に直接大文字で書かれている
  現在: <a href="#about">ABOUT</a>
  修正:  <a href="#about" class="uppercase">About</a>
```

**コントラストの懸念（目視確認のみ）:**
白背景に `text-gray-300`、`text-gray-400` などが使われている場合は警告する。

---

## ディメンション 5: HTML 構造のアクセシビリティチェック

**ナビゲーション:**
- `<nav>` に `aria-label` がない
- アクティブなリンクに `aria-current="page"` がない
- ナビのリンクが `<ul><li>` で囲まれていない

**ボタンとリンク:**
- `<button>` に `type` 属性がない
- アクション用に `<a>` が使われている → `<button>` にすべき

**フォーム:**
- `<input>` に関連する `<label>` がない
- placeholder のみでラベルがない

**見出し階層:**
- `<h1>` 要素が複数ある、またはレベルが飛んでいる

---

## ステップ 3: サマリー

```
## Tailwind CSS レビュー結果

### サマリー
| ディメンション | 件数 |
|-----------|-------|
| クラスの冗長性 | X |
| v4 移行 | X |
| デザイントークン | X |
| アクセシビリティ（Tailwind） | X |
| HTML 構造の a11y | X |
| **合計** | **X** |

### 修正の適用
自動修正できる項目（冗長性、v4 移行、スケール変換）を適用しますか？
```

## 修正の適用

- **レビューのみ:** ユーザーが明示的に修正を依頼しない場合。発見事項を報告するのみで修正は行わない。
- **部分的な修正:** ユーザーが特定ディメンションのみの修正を指定した場合（例：「冗長性だけ直して」）。該当ディメンションの変更リストを表示してユーザーの確認を取ってから適用する。
- **全体修正:** ユーザーが「修正して」「適用して」と明示した場合。すべてのディメンションの変更リストを一覧表示してユーザーの確認を取ってから適用する。ディメンション 2（v4 移行）は手動判断が必要なものを除き確認リストに含める。

---

## 注意事項

- **フレームワーク非依存**: `class=`（HTML/Vue/Astro）と `className=`（React）の両方が対象
- **カスタムクラス**: `@apply` で定義されたクラスは冗長性チェックから除外
- **`@theme` がない場合**: テーマファイルが見つからない場合は報告してデザイントークンチェックをスキップする
