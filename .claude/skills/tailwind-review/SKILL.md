---
name: tailwind-review
description: >
  Tailwind CSS コードのレビュー・最適化・移行、および HTML アクセシビリティチェックを行うスキル。
  以下の場合に必ず使用する：
  · Tailwind クラスを「レビューして」「きれいにして」「確認して」「見て」と依頼された場合
  · `space-y-*`、`space-x-*`、`flex-shrink`、`shadow-sm`、`bg-opacity-*` などの v3 クラスが存在する場合
  · `text-[#1e40af]` や `w-[320px]` のようなハードコードされた任意値が使われている場合
  · 同一要素に `flex` と `block` など、競合・重複クラスが見つかった場合
  · `cn()` / `clsx` の使用可否や `*:` バリアントへの統合を相談された場合
  · v3 から v4 への移行・書き直しを依頼された場合
  · `aria-label`、`ul/li`、`button type`、`label` など HTML アクセシビリティ構造の確認を依頼された場合
  · Tailwind が使われている `.html`、`.tsx`、`.jsx`、`.vue`、`.astro` ファイルのレビューを依頼された場合
  フレームワーク非依存（HTML / React / Vue / Svelte / Astro など）。
  「クラスがなんかおかしい」「v4 にアップグレードしたらデザインが変わった」「CSS モジュールから移行したい」といった曖昧な依頼にも使用する。
---

# Tailwind CSS コードレビュー・最適化スキル

## 概要

このスキルは Tailwind CSS コードを 5 つの観点でレビューし、改善提案または自動修正を行います。

---

## 開始メッセージ

レビューを始める前に、必ず以下のメッセージをユーザーに表示する：

```
> **注意:** 自動修正を適用する場合は、変更を元に戻せる状態にしておいてください。
> Git を使用している場合は `git status` で未コミットの変更を確認してください。
> 未保存の作業がある場合は、`git stash` または `git commit` を先に実行することを推奨します。
```

このメッセージを表示したら、すぐにステップ 0 に進む。返答は待たない。

---

## レビューの実行方法

### ステップ 0: デザインシステム定義の検出

レビューを始める前に、プロジェクトルートに **`DESIGN.md`** が存在するかを確認する。
このファイルにはプロジェクトのデザイントークン（カラー・タイポグラフィ・スペーシング）が定義されている場合があり、ディメンション 3 の提案で最優先参照となる。

```bash
# DESIGN.md の検索
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

### ステップ 0.5: Tailwind バージョンの検出

開始前に、プロジェクトの Tailwind バージョンを検出する。
これによりディメンション 2（v3→v4 移行チェック）の扱いが変わる。

**検出手順（一致したら停止）:**

```bash
# 1. package.json でバージョンを確認
cat package.json | grep '"tailwindcss"'

# 2. tailwind.config.js の有無を確認（v3 の指標）
ls tailwind.config.js 2>/dev/null && echo "found"

# 3. CSS エントリーファイルのインポートスタイルを確認
grep -r "@tailwind\|@import.*tailwindcss" src/ --include="*.css" | head -5
```

**判定基準:**

| 検出結果 | 判定 |
|---|---|
| `"tailwindcss": "^3.*"` または `"tailwindcss": "3.*"` | **v3** |
| `"tailwindcss": "^4.*"` または `"tailwindcss": "4.*"` | **v4** |
| `tailwind.config.js` が存在する | おそらく **v3** |
| CSS に `@tailwind base;` が含まれる | **v3** |
| CSS に `@import "tailwindcss";` が含まれる | **v4** |
| 判定不能 | **不明**（下記参照） |

---

**v3 が検出された場合 → ユーザーに選択肢を提示する:**

```
Tailwind CSS v3 が検出されました。

どのように進めますか？

  1. レビューのみ — 問題を報告するが書き換えはしない
  2. v4 へ移行 — v4 構文に書き直す

どちらをご希望ですか？ (1 / 2)
```

- **選択肢 1（レビューのみ）:** ディメンション 2 は v3 コードの問題のみを報告。書き換えなし。
- **選択肢 2（移行）:** ディメンション 2 の変換テーブルに基づきすべての変換を適用し、ファイルを書き直す。

---

**v4 が検出された場合:**
ディメンション 2 で v3 のパターンが混入していないかを確認する。
v4 プロジェクトで v3 パターンが見つかった場合はバグとして報告する。

---

**不明の場合:**
ユーザーに確認する：
```
Tailwind のバージョンを判定できませんでした。
v3 と v4 のどちらを使用していますか？
```

---

### ステップ 1: 対象ファイルの特定

ファイルが指定されていない場合、プロジェクト内の HTML / JSX / TSX / Vue / Svelte ファイルで Tailwind クラスを含むものを検索する。

```bash
grep -rl "class=" . --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.astro" | grep -v "node_modules" | grep -v "dist"
```

### ステップ 1.5: レビュー範囲の確認（HTML / Astro ファイルのみ）

`.html` / `.astro` ファイルにのみ適用する。
`.tsx` / `.jsx` / `.vue` などのコンポーネントファイルは最初から小さい単位なのでスキップする。

**ファイルを読み込んでサイズを判定する:**
- 80 行以上、または `<section>` / `id` 付き `<div>` が 3 つ以上 → 範囲を確認する
- それより小さいファイル → 確認なしにファイル全体をレビューする

**範囲確認が必要な場合、セクションを自動検出して提示する:**

以下の優先順位でセクション境界を検出する：
1. `<section>` タグ（`id` または `aria-label` の値を使用）
2. トップレベルの `<div id="...">` ブロック
3. `<!-- ... -->` コメントで区切られたブロック
4. セマンティック要素：`<header>` / `<main>` / `<footer>` / `<nav>`

セクションリストをテキストで出力してから `AskUserQuestion` でレビュー範囲を確認する。

**テキスト出力例（AskUserQuestion の前に表示する）:**

```
このファイルは 320 行です。以下のセクションが検出されました：

  1. <header>            — ナビゲーション（1〜30 行）
  2. #hero               — ヒーローセクション（31〜80 行）
  3. #features           — 機能紹介（81〜150 行）
  4. #pricing            — 料金プラン（151〜230 行）
  5. <footer>            — フッター（231〜320 行）
```

**`AskUserQuestion` で範囲を選択する（最大 4 選択肢）:**

`AskUserQuestion` は最大 4 つの選択肢を受け付ける。必ず以下の 4 つを使用する：

```
1. 全体（推奨）  — ページ全体をレビュー
2. 前半         — 上部のセクション
3. 後半         — 下部のセクション
4. 番号で指定   — ユーザーがセクション番号を入力する
```

「番号で指定」が選ばれた場合、次のメッセージで番号を入力してもらう。
選択されたセクションのみ読み込んでレビューする。
「全体」が選ばれた場合はファイル全体をレビューする。

### ステップ 2: 5 つのディメンションでレビューする

**すべての** ディメンションを確認する。1 件でも見つかれば報告する。

---

## ディメンション 1: クラスの冗長性チェック

同一要素上の **競合クラス** または **無意味な重複** を検出する。

**よくあるパターン:**
- レイアウトの競合：`block` と `flex` が共存、`hidden` と `flex` が共存
- 方向の競合：`flex-row` と `flex-col` が共存
- サイズの重複：`w-full` と `w-1/2` が共存
- 表示の重複：`inline-block` と `block` が共存

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
- 大多数が **2 つ以上**の共通クラスを持つ

典型的なパターン：ナビリンク、リストアイテム、フッターカラム。

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
  メリット: クラスを一箇所で管理できる — デザイン変更が 1 回の編集で済む
```

**注意事項:**
- 一部の子要素が異なるクラスを持つ場合（例：アクティブなリンクに `text-blue-600`）、
  共有クラスのみを親に移動し、差分は子に残す：
  ```html
  <ul class="*:text-gray-800 *:transition-colors">
    <li><a>Home</a></li>
    <li><a class="text-blue-600">About（現在のページ）</a></li>  ← 差分を残す
    <li><a>Contact</a></li>
  </ul>
  ```
- `*:` は**直接の子要素のみ**に適用される（孫要素には適用されない）— ネストによる意図しないスコープが発生し得る場合は言及する。
- Tailwind v3.1 以降 / v4 で使用可能。古い v3 プロジェクトでは提案しない。

### cn() / クラスマージ関数の提案（React / TSX / Astro）

`.tsx` / `.jsx` / `.astro` ファイルに対しては、**クラスマージ関数**が使われているかも確認する。
HTML と Vue ファイルではこのチェックをスキップする。

**ステップ 1: 導入状況を確認する**

```bash
# clsx / tailwind-merge / class-variance-authority が package.json にあるか確認
grep -E '"clsx"|"tailwind-merge"|"class-variance-authority"' package.json

# cn() / clsx() がコードで使われているか確認
grep -r "cn(" --include="*.tsx" --include="*.ts" --include="*.astro" src/ | grep -v "node_modules" | head -5
grep -r "clsx(" --include="*.tsx" --include="*.ts" --include="*.astro" src/ | grep -v "node_modules" | head -5
```

**ステップ 2: 状況に応じて対応する**

**パターン A: 未導入（package.json にもコードにも存在しない）**

冗長性の問題が 1 件以上見つかった場合、レビュー結果に以下を追加する：

```
💡 提案: クラスマージ関数の導入

このプロジェクトでは cn() / clsx などのユーティリティが使われていません。
Tailwind では同じプロパティを対象とする複数のクラスがある場合、最後のものが優先されますが、
意図しない上書きが起きることがあります。マージ関数の導入を検討してください。

推奨セットアップ（shadcn/ui スタイル）:
  # ロックファイルからパッケージマネージャーを判定:
  #   package-lock.json → npm install
  #   yarn.lock         → yarn add
  #   pnpm-lock.yaml    → pnpm add
  #   bun.lockb         → bun add
  <pm> add clsx tailwind-merge

  // src/lib/utils.ts
  import { clsx, type ClassValue } from "clsx"
  import { twMerge } from "tailwind-merge"

  export function cn(...inputs: ClassValue[]) {
    return twMerge(clsx(inputs))
  }

使用例:
  // 条件付きクラスを安全に構築
  <div className={cn("flex px-4", isActive && "bg-blue-600", className)}>

  // className プロパティと内部クラスを安全にマージ
  <button className={cn("rounded px-4 py-2", props.className)}>
```

**パターン B: 導入済みだが冗長性が見つかった箇所で使われていない**

```
[冗長性] <ファイル名>:<行>
  問題: `flex` と `block` の両方が指定されている
  現在: <div className="flex block px-4">
  修正:  <div className="flex px-4">
  補足: cn() はすでにこのプロジェクトで設定されています。
        className プロパティを安全にマージするために使用してください：
        <div className={cn("flex px-4", props.className)}>
```

**パターン C: 導入済みで正しく使われている**

報告不要。冗長性の問題が見つかれば通常通り報告する。

---

## ディメンション 2: v3 → v4 移行チェック

Tailwind CSS v4 で非推奨または名称変更されたクラスを検出し、v4 の代替を提案する。

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
| `divide-opacity-50` | `divide-black/50` |
| `ring-opacity-50` | `ring-black/50` |
| `placeholder-opacity-50` | `placeholder-black/50` |

**[名称変更] Flex ユーティリティ**
| v3（旧） | v4（新） |
|---|---|
| `flex-shrink` / `flex-shrink-0` | `shrink` / `shrink-0` |
| `flex-grow` / `flex-grow-0` | `grow` / `grow-0` |

**[名称変更] スケールシフト（⚠️ 見た目がサイレントに変わる）**

v4 ではスケールに `xs` が追加され、既存の名前がすべて 1 段階ずれる。
エラーが出ないため見落としやすい。

| v3 | v4 | 実際のサイズ |
|---|---|---|
| `shadow-sm` | `shadow-xs` | 小さいシャドウ |
| `shadow`（サフィックスなし） | `shadow-sm` | デフォルトシャドウ |
| `blur-sm` | `blur-xs` | 小さいぼかし |
| `blur`（サフィックスなし） | `blur-sm` | デフォルトぼかし |
| `drop-shadow-sm` | `drop-shadow-xs` | 小さいドロップシャドウ |
| `drop-shadow`（サフィックスなし） | `drop-shadow-sm` | デフォルトドロップシャドウ |
| `backdrop-blur-sm` | `backdrop-blur-xs` | 小さいバックドロップぼかし |
| `backdrop-blur`（サフィックスなし） | `backdrop-blur-sm` | デフォルトバックドロップぼかし |
| `rounded-sm` | `rounded-xs` | 小さい角丸 |
| `rounded`（サフィックスなし） | `rounded-sm` | デフォルト角丸 |

> **検出のヒント:** サフィックスなしまたは `-sm` 付きの `shadow`、`blur`、`rounded`、`drop-shadow`、`backdrop-blur` はスケールシフトの候補。報告前にプロジェクトが v4 であることを確認する。

**[変更] アウトライン**
| v3 | v4 |
|---|---|
| `outline-none` | `outline-hidden`（アクセシビリティの明確化のため改名） |
| `outline outline-2` | `outline-2`（`outline` 単体は 1px を設定するようになった） |

**[変更] !important 修飾子の位置**
| v3 | v4 |
|---|---|
| `!flex` `!bg-red-500` `hover:!bg-red-600` | `flex!` `bg-red-500!` `hover:bg-red-600!` |

**[変更] CSS 変数の任意値構文**
| v3 | v4 |
|---|---|
| `bg-[--brand-color]` | `bg-(--brand-color)` |

**[変更] その他**
| v3 | v4 |
|---|---|
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` | `box-decoration-slice` |
| `decoration-clone` | `box-decoration-clone` |

---

### 検出

```bash
# 非推奨クラスを一括で grep
grep -rn \
  -e "space-x-" -e "space-y-" \
  -e "flex-shrink" -e "flex-grow" \
  -e "bg-opacity-" -e "text-opacity-" -e "border-opacity-" \
  -e "ring-opacity-" -e "divide-opacity-" -e "placeholder-opacity-" \
  -e "overflow-ellipsis" -e "outline-none" \
  -e "\!flex\b" -e "\!block\b" -e "\!hidden\b" \
  --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.astro" \
  . | grep -v "node_modules" | grep -v "dist"
```

スケールシフト（shadow/blur/rounded）は `shadow\b`、`blur\b`、`rounded\b`、`shadow-sm\b` などで grep して目視確認する。

---

### 報告フォーマット

```
[v4 移行] <ファイル名>:<行>
  問題: `shadow-sm` は v4 で `shadow-xs` に名称変更（スケールシフト）
  現在: class="shadow-sm rounded px-4"
  修正:  class="shadow-xs rounded-sm px-4"
  補足: shadow（サフィックスなし）→ shadow-sm、rounded（サフィックスなし）→ rounded-sm も確認してください
```

---

### 移行の実行（ステップ 0.5 で「2. v4 へ移行」を選んだ場合のみ）

報告するだけでなく、ファイルを書き直す。以下の手順に従う：

**1. 変換リストを作成する**

すべての対象ファイルをスキャンし、必要な変更をリストアップしてから、適用前にユーザーへ確認する：

```
以下の変換を適用します（計 X 件）：

  [1] 12 行目  space-y-4        → flex flex-col gap-4
  [2] 18 行目  flex-shrink-0    → shrink-0
  [3] 24 行目  shadow-sm        → shadow-xs
  [4] 24 行目  rounded          → rounded-sm
  [5] 31 行目  bg-opacity-50    → bg-black/50
  [6] 45 行目  outline-none     → outline-hidden

進めますか？（yes / 番号で除外例: "3, 4 をスキップ"）
```

**2. 承認後に Edit ツールで適用する**

ファイルを直接編集する。変更ごとに確認する必要はない — リストを確認してから一括で適用する。

**3. 手動判断が必要な変換（自動適用しない）**

以下は文脈に依存するため、提案に留めてユーザーに判断を委ねる：

- `space-y-*` / `space-x-*` → 変換先は親要素にすでに `flex` があるかに依存する
- `shadow`（サフィックスなし） → v3 で意図的だったか確認が必要（デザインレビューが必要）
- `!important` 修飾子（`!flex` → `flex!`） → 大量にある場合は一括置換でエラーが起きやすい

**4. CSS エントリーファイルの更新も提案する**

v3 → v4 移行では CSS の構文も変わる。ファイルを確認して提案する：

```
[移行] src/style.css
  現在:
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
  修正:
    @import "tailwindcss";

この変更を適用しますか？
```

---

## ディメンション 3: デザイントークンの使用チェック

**任意値**を検出し、`@theme` 変数または Tailwind 標準スケール値への置き換えを提案する。

このディメンションは、Figma の正確な寸法を Tailwind スケールではなく任意値として再現しがちな **Figma MCP** が生成したコードに特に重要。

**任意値の例:**
- `text-[#294779]` → `text-primary`（`@theme` に `--color-primary: #294779` があれば）
- `bg-[#1a1a1a]` → `bg-dark`
- `w-[320px]` → `w-80`（Tailwind スケールで表現できれば）
- `font-[600]` → `font-semibold`

### 参照の優先順位

任意値の置き換えを提案する際は、以下の順に参照し、最初に一致したものを使用する：

| 優先度 | 参照元 | 使い方 |
|---|---|---|
| **1** | `DESIGN.md` | プロジェクトチームが定義したトークン名と値。そのまま使用する。 |
| **2** | CSS エントリーファイルの `@theme` | Tailwind ユーティリティにコンパイルされる CSS カスタムプロパティ（`--color-*`、`--font-*` など） |
| **3** | Tailwind 標準スケール | 標準ユーティリティ（`text-base`、`gray-700`、`font-semibold` など）（3-B / 3-C テーブル参照） |
| **4** | 任意値のまま維持 | 一致なし — 値を維持してデザインシステムへの追加を提案 |

提案が `DESIGN.md` 由来の場合、報告にソースを明記する：

```
[トークン] <ファイル名>:<行>
  問題: ハードコードされた色 `[#1a3a5c]` は DESIGN.md で定義されたトークンと一致
  現在: class="text-[#1a3a5c]"
  修正:  class="text-brand-dark"  # DESIGN.md で brand-dark: #1a3a5c として定義されている
```

**手順:**
1. `DESIGN.md` トークン（ステップ 0 で読み込み済み）を確認 — 完全一致を優先
2. `src/style.css`（または `globals.css` など）の `@theme` ブロックを読み込む — CSS 変数と照合
3. 1 または 2 で一致しない場合、Tailwind 標準スケールテーブル（3-B / 3-C）を確認
4. どこにも一致しない場合は「デザイントークンが未定義」としてフラグを立て、定義を提案

**報告フォーマット:**
```
[トークン] <ファイル名>:<行>
  問題: ハードコードされた色 `[#294779]`
  現在: class="text-[#294779]"
  提案:  class="text-primary"  # style.css に --color-primary: #294779 が定義されている
```

スケール変換を積極的に提案する：
```
[トークン] <ファイル名>:<行>
  問題: w-[320px] は w-80（320px）で表現できる
  現在: class="w-[320px]"
  修正:  class="w-80"
```

---

### 3-A: フォントファミリーの任意値検出

Figma MCP はフォント名を任意値として直接書くことが多い。このパターンを検出して `@theme` への定義または標準ユーティリティの使用を提案する。

**検出対象:**
- `font-['Inter',_sans-serif]`
- `font-['Noto_Sans_JP']`
- `font-[Inter]`
- 文字列（数値以外）を含む `font-[`

**手順:**
1. 対象ファイルで `font-\[` を grep する
2. 各一致箇所について、`@theme` に対応する `--font-*` 変数があるか確認する
3. 組み込みユーティリティ（`font-sans`、`font-serif`、`font-mono`）にマップできるか確認する

**報告フォーマット — テーマ変数が未定義の場合:**
```
[トークン] <ファイル名>:<行>
  問題: フォントファミリーが任意値 `font-['Inter',_sans-serif]` として書かれている
  現在: class="font-['Inter',_sans-serif]"
  提案:
    1. @theme で定義する:
         @theme {
           --font-sans: 'Inter', sans-serif;
         }
       その後: class="font-sans"

    2. または組み込みユーティリティにマップ: font-sans / font-serif / font-mono
       （代替前に意図したフォントを確認する）
```

**報告フォーマット — テーマ変数がすでに存在する場合:**
```
[トークン] <ファイル名>:<行>
  問題: フォントファミリーが任意値 `font-['Inter',_sans-serif]` として書かれている
  現在: class="font-['Inter',_sans-serif]"
  修正:  class="font-sans"  # style.css に --font-sans: 'Inter', sans-serif が定義されている
```

---

### 3-B: ピクセル / 単位スケールのマッピング

以下のテーブルを使用して、任意の px / em / rem 値を Tailwind スケールユーティリティに変換する。
最も近いものを提案する。値がステップの間にある場合は両隣を提示してユーザーに確認を求める。

**フォントサイズ（`text-[*]`）**

| 任意値 | Tailwind ユーティリティ | 実際のサイズ |
|---|---|---|
| `text-[10px]` | `text-xs` | 0.75rem / 12px |
| `text-[12px]` | `text-xs` | 0.75rem / 12px |
| `text-[14px]` | `text-sm` | 0.875rem / 14px |
| `text-[16px]` | `text-base` | 1rem / 16px |
| `text-[18px]` | `text-lg` | 1.125rem / 18px |
| `text-[20px]` | `text-xl` | 1.25rem / 20px |
| `text-[24px]` | `text-2xl` | 1.5rem / 24px |
| `text-[30px]` | `text-3xl` | 1.875rem / 30px |
| `text-[36px]` | `text-4xl` | 2.25rem / 36px |
| `text-[48px]` | `text-5xl` | 3rem / 48px |
| `text-[60px]` | `text-6xl` | 3.75rem / 60px |
| `text-[72px]` | `text-7xl` | 4.5rem / 72px |

**行の高さ（`leading-[*]`）**

| 任意値 | Tailwind ユーティリティ | 値 |
|---|---|---|
| `leading-[1]` | `leading-none` | 1 |
| `leading-[1.25]` | `leading-tight` | 1.25 |
| `leading-[1.375]` | `leading-snug` | 1.375 |
| `leading-[1.5]` | `leading-normal` | 1.5 |
| `leading-[1.625]` | `leading-relaxed` | 1.625 |
| `leading-[2]` | `leading-loose` | 2 |

**文字間隔（`tracking-[*]`）**

| 任意値 | Tailwind ユーティリティ | 値 |
|---|---|---|
| `tracking-[-0.05em]` | `tracking-tighter` | -0.05em |
| `tracking-[-0.025em]` | `tracking-tight` | -0.025em |
| `tracking-[0em]` | `tracking-normal` | 0em |
| `tracking-[0.025em]` | `tracking-wide` | 0.025em |
| `tracking-[0.05em]` | `tracking-wider` | 0.05em |
| `tracking-[0.1em]` | `tracking-widest` | 0.1em |

**フォントウェイト（`font-[*]`）**

| 任意値 | Tailwind ユーティリティ |
|---|---|
| `font-[100]` | `font-thin` |
| `font-[200]` | `font-extralight` |
| `font-[300]` | `font-light` |
| `font-[400]` | `font-normal` |
| `font-[500]` | `font-medium` |
| `font-[600]` | `font-semibold` |
| `font-[700]` | `font-bold` |
| `font-[800]` | `font-extrabold` |
| `font-[900]` | `font-black` |

**スペーシング / サイズ — よく使う値（`w-[*]`、`h-[*]`、`p-[*]`、`m-[*]`、`gap-[*]` など）**

| px 値 | Tailwind スケール | rem |
|---|---|---|
| 4px | `1` | 0.25rem |
| 8px | `2` | 0.5rem |
| 12px | `3` | 0.75rem |
| 16px | `4` | 1rem |
| 20px | `5` | 1.25rem |
| 24px | `6` | 1.5rem |
| 32px | `8` | 2rem |
| 40px | `10` | 2.5rem |
| 48px | `12` | 3rem |
| 64px | `16` | 4rem |
| 80px | `20` | 5rem |
| 96px | `24` | 6rem |
| 128px | `32` | 8rem |
| 160px | `40` | 10rem |
| 192px | `48` | 12rem |
| 224px | `56` | 14rem |
| 256px | `64` | 16rem |
| 288px | `72` | 18rem |
| 320px | `80` | 20rem |
| 384px | `96` | 24rem |

> **ヒント:** Tailwind のスペーシングスケールは `1 単位 = 4px`。px を 4 で割るとスケール番号になる。
> 結果が整数であれば標準ユーティリティが存在する。そうでなければ任意値を維持する。

**報告フォーマット:**
```
[トークン] <ファイル名>:<行>
  問題: text-[16px] は text-base（16px / 1rem）で表現できる
  現在: class="text-[16px] leading-[1.5] tracking-[0.05em]"
  修正:  class="text-base leading-normal tracking-wider"
```

---

### 3-C: Tailwind 組み込みカラーのマッチング

`@theme` 変数が定義されていなくても、ハードコードされた hex / rgb カラー値を **Tailwind 組み込みカラーパレット**と照合して最も近い色を提案する。

**対象:** `text-[*]`、`bg-[*]`、`border-[*]`、`ring-[*]`、`fill-[*]`、`stroke-[*]`、`shadow-[*]`、`divide-[*]`、`outline-[*]`

**照合アプローチ:**
1. 任意クラスから hex 値を抽出する
2. 以下の参照テーブルと照合する（完全一致を優先、次に hex 距離で最近傍）
3. 近似一致の場合は「（近似値）」としてユーザーへ目視確認を求める

**Tailwind カラー参照（hex → ユーティリティ）**

| Hex | ユーティリティ |
|---|---|
| `#f9fafb` | `gray-50` |
| `#f3f4f6` | `gray-100` |
| `#e5e7eb` | `gray-200` |
| `#d1d5db` | `gray-300` |
| `#9ca3af` | `gray-400` |
| `#6b7280` | `gray-500` |
| `#4b5563` | `gray-600` |
| `#374151` | `gray-700` |
| `#1f2937` | `gray-800` |
| `#111827` | `gray-900` |
| `#eff6ff` | `blue-50` |
| `#dbeafe` | `blue-100` |
| `#bfdbfe` | `blue-200` |
| `#93c5fd` | `blue-300` |
| `#60a5fa` | `blue-400` |
| `#3b82f6` | `blue-500` |
| `#2563eb` | `blue-600` |
| `#1d4ed8` | `blue-700` |
| `#1e40af` | `blue-800` |
| `#1e3a8a` | `blue-900` |
| `#fef2f2` | `red-50` |
| `#fee2e2` | `red-100` |
| `#fca5a5` | `red-300` |
| `#f87171` | `red-400` |
| `#ef4444` | `red-500` |
| `#dc2626` | `red-600` |
| `#b91c1c` | `red-700` |
| `#f0fdf4` | `green-50` |
| `#dcfce7` | `green-100` |
| `#86efac` | `green-300` |
| `#4ade80` | `green-400` |
| `#22c55e` | `green-500` |
| `#16a34a` | `green-600` |
| `#15803d` | `green-700` |
| `#fefce8` | `yellow-50` |
| `#fef9c3` | `yellow-100` |
| `#fde047` | `yellow-300` |
| `#facc15` | `yellow-400` |
| `#eab308` | `yellow-500` |
| `#ca8a04` | `yellow-600` |
| `#ffffff` | `white` |
| `#000000` | `black` |
| `transparent` | `transparent` |

> テーブルにない色は、視覚的な hex 近傍で最も近い色を使用し、近似値としてマークする。
> 注意: Tailwind v4 には拡張パレット（slate、zinc、stone、sky、indigo、violet、purple、fuchsia、pink、rose、emerald、teal、cyan、lime、amber、orange）も含まれる — 同じ照合ロジックを適用する。

**報告フォーマット — 完全一致:**
```
[トークン] <ファイル名>:<行>
  問題: ハードコードされた色 `[#374151]` は Tailwind 組み込みカラーと一致
  現在: class="text-[#374151]"
  修正:  class="text-gray-700"
```

**報告フォーマット — 近似一致:**
```
[トークン] <ファイル名>:<行>
  問題: `[#3a4050]` は gray-700（#374151）に近い — 目視で確認してください
  現在: class="text-[#3a4050]"
  提案:  class="text-gray-700"  ← 近似値；適用前に確認してください
```

---

### 3-D: 簡略化できる任意値（v4）

Tailwind v4 では、一部の任意値がブラケットなしのネイティブユーティリティで書ける。
これを検出して短縮形を提案する。リンターの警告（例: Biome）を防ぎ、可読性を向上させる。

**検出対象と変換:**

| パターン | 例 | 修正 |
|---|---|---|
| `aspect-[N/M]` | `aspect-[16/7]` | `aspect-16/7` |
| `aspect-[N/M]` | `aspect-[4/3]` | `aspect-4/3` |
| `w-[N/M]` | `w-[1/2]` | `w-1/2` |
| `w-[N/M]` | `w-[2/3]` | `w-2/3` |

`w-[N/M]` → `w-N/M` は N と M がどちらも整数の場合のみ適用する。
`w-[320px]`、`w-[calc(...)]` などその他の任意値はそのまま維持する。

**検出:**

```bash
grep -rn -e "aspect-\[" -e "w-\[[0-9]" --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.astro" . | grep -v "node_modules" | grep -v "dist"
```

> **注意:** ブラケット内の `N/M` 比率パターンのみ簡略化する。その他の任意値はそのまま維持する。

**報告フォーマット:**
```
[簡略化] <ファイル名>:<行>
  問題: `aspect-[16/7]` は Tailwind v4 で `aspect-16/7` と書ける
  現在: class="... aspect-[16/7] ..."
  修正:  class="... aspect-16/7 ..."

[簡略化] <ファイル名>:<行>
  問題: `w-[1/2]` は `w-1/2` と書ける
  現在: class="... w-[1/2] ..."
  修正:  class="... w-1/2 ..."
```

---

## ディメンション 4: アクセシビリティチェック（Tailwind）

視覚的なスタイリングの選択がアクセシビリティを損なっていないか確認する。

**主なチェック項目:**

**大文字テキスト:**
HTML に直接大文字で書かれているテキストを検出する。スクリーンリーダーが一文字ずつ読み上げる場合がある。
視覚的に大文字にするには `uppercase` クラスを使用する。

```
[a11y] <ファイル名>:<行>
  問題: テキストが HTML に直接大文字で書かれている
  現在: <a href="#about">ABOUT</a>
  修正:  <a href="#about" class="uppercase">About</a>
```

**コントラストの懸念（目視確認のみ）:**
白または明るい背景に `text-gray-300`、`text-gray-400` などの薄いグレーテキストが使われている場合は警告する。
自動検出はしない — 注意を促すのみ。

```
[a11y] <ファイル名>:<行>
  警告: `text-gray-300` は白背景でコントラストが不足している可能性がある
  対応: WCAG AA のコントラスト比（4.5:1）を満たしているか確認してください
```

**フォームのアクセシビリティ:**
入力欄やボタンのラベルを視覚的に隠すために `sr-only` ではなく `hidden` が使われていないか確認する。

---

## ディメンション 5: HTML 構造のアクセシビリティチェック

Tailwind クラスとは独立して、HTML 要素のセマンティクス・構造・ARIA 属性が正しいかを確認する。
ナビゲーション・フォーム・インタラクティブ要素でよく問題が発生する。

**主なチェック項目:**

**ナビゲーション:**
- `<nav>` に `aria-label` がない（複数の nav がある場合は必須、1 つでも有益）
- アクティブなリンクに `aria-current="page"` がない
- ナビのリンクが `<ul><li>` で囲まれていない（スクリーンリーダーがリスト項目数を読み上げられない）

```
[HTML a11y] <ファイル名>:<行>
  問題: <nav> に aria-label がない
  現在: <nav class="flex ...">
  修正:  <nav aria-label="メインナビゲーション" class="flex ...">
```

**ボタンとリンク:**
- `<button>` に `type` 属性がない（フォーム内では `type="submit"` がデフォルトになり、意図しない送信が起きる場合がある）
- ナビゲーションではなくアクション用に `<a>` が使われている → `<button>` にすべき
- `role="button"` も意味のある `href` もない `<a href="#">`

```
[HTML a11y] <ファイル名>:<行>
  問題: <button> に type 属性がない
  現在: <button class="bg-blue-600 ...">LOGIN</button>
  修正:  <button type="button" class="bg-blue-600 ...">Login</button>
```

**画像:**
- `<img>` に `alt` 属性がない、または空文字列（空の `alt=""` は装飾画像に正しい；意味のある画像には説明が必要）

**フォーム:**
- `<input>` に関連する `<label>` がない（`id` / `for` のバインディングがない）
- `<input type="text">` が `<label>` なしで `placeholder` のみを使っている（placeholder はラベルの代替にならない）

```
[HTML a11y] <ファイル名>:<行>
  問題: <input> に関連する <label> がない
  現在: <input type="email" placeholder="your@email.com" class="...">
  修正:  <label for="email">メールアドレス</label>
         <input id="email" type="email" placeholder="your@email.com" class="...">
```

**見出し階層:**
- `<h1>` 要素が複数ある、または見出しレベルが飛んでいる（例：`<h1>` から `<h3>` に直接進む）

---

## ステップ 3: サマリー

すべてのディメンションを確認したら、以下のフォーマットで結果を提示する：

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

### 詳細
（各ディメンションの発見事項を列挙）

### 修正の適用
自動修正できる項目（冗長性、v4 移行、スケール変換）を適用しますか？
```

## 修正の適用

- ユーザーが「修正して」「適用して」と明示した場合 → ファイルを直接編集する
- レビューのみの場合 → 提案を提示して確認を求める
- 部分的な修正の場合 → 適用する項目を確認してから進める

---

## 注意事項

- **フレームワーク非依存**: `class=`（HTML/Vue/Astro）と `className=`（React）の両方が対象
- **カスタムクラス**: `@apply` で定義されたクラスは Tailwind ユーティリティではないため冗長性チェックから除外
- **文脈が重要**: `space-y-*` でも意図的な使用の場合がある — v4 への移行を強く推奨するが影響を説明する
- **`@theme` がない場合**: テーマファイルが見つからない場合は報告してデザイントークンチェックをスキップする
