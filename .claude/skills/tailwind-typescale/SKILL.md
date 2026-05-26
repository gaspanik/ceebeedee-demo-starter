---
name: tailwind-typescale
description: >
  ベースフォントサイズとスケール比率名（typescale.com 準拠）を選択して、
  Tailwind CSS v4 プロジェクトに調和のとれたタイプスケールを適用する。
  `@theme` に `--text-xs` ～ `--text-9xl` を設定し（Tailwind の全テキストサイズクラスに対応）、
  `@layer base` で h1〜h6 のフォントサイズをマッピングする。
  以下の場合に使用する：
  "タイプスケール変えたいんだけど"、
  "フォントサイズを整えたい"、
  "見出しのサイズをちゃんとしたい"、
  "タイポグラフィのスケールを設定して"、
  "apply a type scale"、"set up typography scale"、"change font sizes"
argument-hint: "[base-size] [scale-name]"
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

# タイプスケールの適用

Tailwind CSS v4 プロジェクトに調和のとれたタイポグラフィスケールを設定します。

---

## ステップ 0: 事前確認

ユーザーに何かを聞く前に、すべての確認を実行する。

```bash
# 1. Tailwind CSS v4 エントリーファイルを検索
#    主要シグナル: @import "tailwindcss" または @import 'tailwindcss'
grep -rl --include="*.css" --exclude-dir=node_modules --exclude-dir=dist '@import ["'"'"']tailwindcss' . 2>/dev/null

# 2. DESIGN.md の確認
ls DESIGN.md 2>/dev/null || echo "not found"

# 3. tailwind.config.js の確認（v3 の指標）
ls tailwind.config.js 2>/dev/null && echo "v3 config found" || echo "no v3 config"
```

### ステップ 0-A: 対象 CSS ファイルの決定

**ファイルが 1 つだけ見つかった場合:** それを対象として使用する。ステップ 0-B へ進む。

**複数見つかった場合:** `AskUserQuestion` でどのファイルにスケールを適用するか確認する（各パスを選択肢として列挙）。選択されたファイルを対象として使用する。

**見つからなかった場合:** `@theme` ブロックをフォールバック検索する：

```bash
grep -rl --include="*.css" --exclude-dir=node_modules --exclude-dir=dist '@theme' . 2>/dev/null
```

同じ 1 件 / 複数 / なし のロジックを適用する。

**それでも見つからない場合:** Tailwind CSS エントリーファイル（`@import "tailwindcss"` が含まれるファイル）のパスをユーザーに尋ねて停止する。

### ステップ 0-B: 確認内容を報告する

対象ファイル内の既存の `--text-*` 変数を確認する：

```bash
grep -n "\-\-text-" {対象ファイル} 2>/dev/null || echo "none"
```

見つかった内容のサマリーを表示する：

```
## 現在のプロジェクト状態

対象 CSS:            {path/to/file.css}
DESIGN.md:           あり / なし
tailwind.config.js:  あり（v3） / なし（v4）

タイプスケール（--text-*）:
  見つかりました:
    --text-xs:   0.600rem  （既存）
    --text-sm:   0.750rem  （既存）
    ... （全変数を列挙）
  見つかりません
```

**`DESIGN.md` が存在する場合:**
ファイルを読み込んでタイポグラフィ関連の内容（フォントサイズ、タイプスケール、スケール名など）を確認する。
見つかった場合は「DESIGN.md にタイポグラフィ設定が含まれています：」として関連箇所を表示する。

**`tailwind.config.js` が存在する場合（v3）:**
このスキルは Tailwind CSS v4 が必要。警告して停止する：

```
⚠️  tailwind.config.js が検出されました — このプロジェクトは Tailwind CSS v3 を使用しているようです。
このスキルは v4（@theme ディレクティブと --text-* 変数）が必要です。
v3 プロジェクトの場合は tailwind.config.js の theme.fontSize でフォントサイズを設定してください。
```

### ステップ 0-C: --text-* 変数が存在する場合の上書き確認

1 つ以上の `--text-*` 変数が見つかった場合、`AskUserQuestion` で確認する：

- 新しいスケールで上書きする
- キャンセル（既存を維持する）

キャンセルの場合は停止する。
確認された場合はステップ 1 へ進む。

`--text-*` 変数が存在しない場合は直接ステップ 1 へ進む。

---

## ステップ 1: パラメーターの収集

`$ARGUMENTS` を以下のように解釈する：

- 引数なし → ステップ 1-A（AskUserQuestion で確認）
- 引数あり → スペース区切りで `[base-size] [scale-name]` としてパース
  - 例: `16px perfect-fourth`、`18 golden-ratio`
  - 不足している値はステップ 1-A で確認する

### ステップ 1-A: AskUserQuestion で確認する

両方の質問を **同時に** 1 回の `AskUserQuestion` 呼び出しで提示する。

**Q1: ベースフォントサイズ**
- 16px – 標準（推奨）
- 14px – コンパクト
- 18px – ゆとりあり
- 20px – 大きめ

**Q2: タイプスケール**
- Perfect Fourth 1.333 – バランスが良く読みやすい（推奨）
- Major Third 1.250 – 穏やか・コンパクト
- Golden Ratio 1.618 – ダイナミックなコントラスト
- Minor Third 1.200 – 繊細・洗練

両方の回答後、ステップ 2 へ進む。

---

## ステップ 2: スケールの計算

以下の計算式を使用する：

```
base_rem = ベースフォントサイズ（px） / 16
ratio    = 選択したスケール比率

text-xs   = base_rem / ratio^2
text-sm   = base_rem / ratio^1
text-base = base_rem
text-lg   = base_rem * ratio^1
text-xl   = base_rem * ratio^2
text-2xl  = base_rem * ratio^3
text-3xl  = base_rem * ratio^4
text-4xl  = base_rem * ratio^5
text-5xl  = base_rem * ratio^6
text-6xl  = base_rem * ratio^7
text-7xl  = base_rem * ratio^8
text-8xl  = base_rem * ratio^9
text-9xl  = base_rem * ratio^10
```

各値は **小数点以下 3 桁** に丸める（例: `1.333rem`）。

### 各ステップの推奨行間（`leading-*` ユーティリティ）

> **重要:** `--text-*` 変数に行間を埋め込まない。Tailwind v4 では `--text-base: 1rem 1.5` と書くと `font-size: 1rem 1.5` という無効な CSS が生成され、ブラウザが無視して 16px にフォールバックしてしまう。rem 値のみを使用すること: `--text-base: 1rem`。

行間は要素ごとに Tailwind の `leading-*` ユーティリティで制御する：

| ステップ | 推奨 leading-* |
|---|---|
| text-xs / text-sm | `leading-relaxed`（1.625） |
| text-base / text-lg | `leading-normal`（1.5） |
| text-xl / text-2xl | `leading-snug`（1.375） |
| text-3xl | `leading-snug`（1.375） |
| text-4xl / text-5xl | `leading-tight`（1.25） |
| text-6xl 以上 | `leading-none`（1.0） |

---

## ステップ 3: プレビューの表示と確認

計算結果をテキストで表示する（ここでは AskUserQuestion を使わない）：

```
## タイプスケール プレビュー

設定: ベース {Xpx} × {スケール名}（{ratio}）

| Tailwind クラス | サイズ（rem） | サイズ（px） |
|---|---|---|
| text-xs   | X.XXXrem | XX.Xpx |
| text-sm   | X.XXXrem | XX.Xpx |
| text-base | X.XXXrem | XX.Xpx |
| text-lg   | X.XXXrem | XX.Xpx |
| text-xl   | X.XXXrem | XX.Xpx |
| text-2xl  | X.XXXrem | XX.Xpx |
| text-3xl  | X.XXXrem | XX.Xpx |
| text-4xl  | X.XXXrem | XX.Xpx |
| text-5xl  | X.XXXrem | XX.Xpx |
| text-6xl  | X.XXXrem | XX.Xpx |
| text-7xl  | X.XXXrem | XX.Xpx |
| text-8xl  | X.XXXrem | XX.Xpx |
| text-9xl  | X.XXXrem | XX.Xpx |

h1〜h6 のマッピング:
  h1 → text-5xl（{size}rem）
  h2 → text-4xl（{size}rem）
  h3 → text-3xl（{size}rem）
  h4 → text-2xl（{size}rem）
  h5 → text-xl （{size}rem）
  h6 → text-lg （{size}rem）
```

次に `AskUserQuestion` で確認する：
- このスケールを適用する（推奨）
- 別のスケールを試す
- キャンセル

「別のスケールを試す」→ ステップ 1 に戻る。
「キャンセル」→ 停止する。

---

## ステップ 3.5: バックアップ確認

変更を書き込む前に実行する：

```bash
git status --short 2>/dev/null || echo "not a git repo"
```

- **未コミットの変更がある Git リポジトリ:** `{対象ファイル}` が上書きされることを警告する。先に `git stash` または `git commit` を実行することを推奨し、`AskUserQuestion` で確認する：
  - このまま続ける
  - キャンセル（先に stash/commit する）

  キャンセルの場合は停止する。

- **Git リポジトリ、クリーンな作業ツリー:** 何も言わずに続行する。
- **Git リポジトリでない場合:** バージョン管理がないことを警告し、続行前に `{対象ファイル}` の手動バックアップを推奨する。`AskUserQuestion` で確認する：
  - このまま続ける
  - キャンセル（先にバックアップする）

  キャンセルの場合は停止する。

---

## ステップ 4: 対象 CSS ファイルを編集する

検出した対象ファイルの `@theme` ブロックに `--text-*` 変数を追加または上書きする。

### ルール

- 既存の `--text-*` 変数がある場合は上書きする
- ない場合は `@theme` の閉じ `}` の前に追記する
- `@theme` ブロックが存在しない場合は新しく作成する

### 出力例

```css
  /* タイプスケール: {スケール名}（{ratio}）— ベース {Xpx} */
  --text-xs: X.XXXrem;
  --text-sm: X.XXXrem;
  --text-base: X.XXXrem;
  --text-lg: X.XXXrem;
  --text-xl: X.XXXrem;
  --text-2xl: X.XXXrem;
  --text-3xl: X.XXXrem;
  --text-4xl: X.XXXrem;
  --text-5xl: X.XXXrem;
  --text-6xl: X.XXXrem;
  --text-7xl: X.XXXrem;
  --text-8xl: X.XXXrem;
  --text-9xl: X.XXXrem;
```

将来の参照のためにスケール名とベースサイズを記録するコメントを含める。

### h1〜h6 フォントサイズのマッピング

`@layer base` 内の `h1`〜`h6` ブロックに `font-size` 宣言を追加する。

`@layer base` にすでに `h1`〜`h6` が存在する場合は、各ルールに `font-size: var(--text-Nxl)` を追加する。
存在しない場合は新しいルールセットを追加する：

```css
@layer base {
  h1 { font-size: var(--text-5xl); }
  h2 { font-size: var(--text-4xl); }
  h3 { font-size: var(--text-3xl); }
  h4 { font-size: var(--text-2xl); }
  h5 { font-size: var(--text-xl); }
  h6 { font-size: var(--text-lg); }
}
```

既存の `h1`〜`h6` ブロックに `font-family` などのプロパティがある場合はそれを維持し、`font-size` のみを追加する。

---

## ステップ 4.5: h1〜h6 の明示的なテキストサイズクラスの削除

スケールを適用した後、明示的な `text-*` サイズクラスを持つ見出し要素を HTML ファイルからスキャンする。
これらは `@layer base` のマッピングを上書きするため、新しいスケールが視覚的に反映されない。

### 検出

```bash
grep -rn "<h[1-6][^>]*class=\"[^\"]*text-\(xs\|sm\|base\|lg\|xl\|2xl\|3xl\|4xl\|5xl\|6xl\|7xl\|8xl\|9xl\)" \
  --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" \
  . | grep -v "node_modules" | grep -v "dist"
```

### 候補リスト

一致するものが見つかった場合、確認を求める前にテキストで表示する：

```
以下の h1〜h6 要素にスケールマッピングを上書きする明示的な text-* サイズクラスがあります：

  [1] post.html:68   <h2 class="text-ink font-semibold text-xl mt-10 mb-4">
                      → text-xl を削除（h2 は @layer base で text-4xl を使用）
  [2] index.html:22  <h2 class="mt-2 font-medium text-primary text-base">
                      → text-base を削除（h2 は @layer base で text-4xl を使用）
```

次に `AskUserQuestion` で確認する：
- 一覧のクラスをすべて削除する（推奨）
- 個別に選択する
- スキップ（すべてそのまま維持する）

「個別に選択する」→ 各項目を番号付きで列挙し、削除する番号を入力してもらう。
「スキップ」→ 変更なしでステップ 4.6 へ進む。

### 削除の適用

`class` 属性から `text-*` サイズクラストークンのみを削除する。要素の他のクラスはすべて維持する。

> **注意:** `text-*` カラークラス（例: `text-ink`、`text-muted`、`text-primary`）は削除しない。
> 削除対象はサイズユーティリティのみ：`text-xs`、`text-sm`、`text-base`、`text-lg`、`text-xl`、`text-2xl` … `text-9xl`。

---

## ステップ 4.6: DESIGN.md タイポグラフィセクションの更新

プロジェクトルートに `DESIGN.md` が存在する場合、そのタイポグラフィセクションが新しいスケールとズレている可能性がある。

現在の `DESIGN.md` を読み込んで、YAML フロントマターの `typography:` ブロックにフォントサイズの値が含まれているか確認する。含まれている場合は表示する：

```
DESIGN.md にフォントサイズを参照するタイポグラフィトークンが含まれています。
新しいスケールではこれらの値が変わります。DESIGN.md を更新しますか？
```

`AskUserQuestion` で確認する：
- DESIGN.md を更新する（推奨）
- スキップ

「DESIGN.md を更新する」の場合：
1. 新しいスケールを使用して関連する見出しレベルの px 値を再計算する（h1 → text-5xl、h2 → text-4xl など）
2. `typography:` YAML ブロックの `fontSize` 値を新しいスケールに合わせて更新する
3. `## Typography` マークダウンセクションの本文を新しいスケール名と比率に合わせて更新する

DESIGN.md にフォントサイズの値が含まれていない場合はサイレントにスキップする。

---

## ステップ 5: 完了

報告前にロックファイルからパッケージマネージャーを検出する：

```bash
ls pnpm-lock.yaml yarn.lock package-lock.json bun.lockb 2>/dev/null | head -1
```

| ロックファイル | パッケージマネージャー |
|---|---|
| `pnpm-lock.yaml` | `pnpm` |
| `yarn.lock` | `yarn` |
| `package-lock.json` | `npm` |
| `bun.lockb` | `bun` |

検出したパッケージマネージャーを完了メッセージで使用する：

```
## 適用完了

{スケール名}（ベース {Xpx}、比率 {ratio}）を {target file} に適用しました。

変更内容:
  - @theme: --text-xs ～ --text-9xl を設定（rem 値のみ）
  - @layer base: h1〜h6 のフォントサイズをスケールにマッピング
  - HTML: X 個の見出し要素から明示的な text-* を削除  ← 削除した場合
  - DESIGN.md: タイポグラフィセクションを更新             ← 更新した場合

開発サーバーが起動中であれば変更はすぐに反映されます。
起動するには: <pm> run dev

💡 上記の行間はデフォルト値です。要素ごとに Tailwind の leading-* ユーティリティで上書きできます：
   leading-tight（1.25）/ leading-snug（1.375）/ leading-normal（1.5）
   leading-relaxed（1.625）/ leading-loose（2）
```

---

## 注意事項

- このスキルは Tailwind CSS **v4** 専用（`@theme` ディレクティブと `--text-*` 変数が必要）
- v3 プロジェクトの場合は `tailwind.config.js` の `theme.fontSize` でフォントサイズを設定する
- Tailwind の全テキストサイズクラスに対応：`text-xs` ～ `text-9xl`
