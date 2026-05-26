---
description: ベースフォントサイズとスケール比率を選択して、Tailwind CSS v4 プロジェクトに調和のとれたタイプスケールを適用する。@theme に --text-xs〜--text-9xl を設定し、@layer base で h1〜h6 のフォントサイズをマッピングする
tools:
  - search/codebase
  - edit/editFiles
  - execute/getTerminalOutput
  - execute/runInTerminal
---

ベースフォントサイズとスケール比率名（typescale.com 準拠）を選択して、Tailwind CSS v4 プロジェクトに調和のとれたタイポグラフィスケールを設定します。

**返答言語:** 日本語で返答する。

---

## ステップ 0: 事前確認

ユーザーに何かを聞く前に、すべての確認を実行する。

```bash
# 1. Tailwind CSS v4 エントリーファイルを検索（シングル／ダブルクォート両対応）
grep -rl --include="*.css" --exclude-dir=node_modules --exclude-dir=dist "@import 'tailwindcss'" . 2>/dev/null
grep -rl --include="*.css" --exclude-dir=node_modules --exclude-dir=dist '@import "tailwindcss"' . 2>/dev/null

# 2. DESIGN.md の確認
ls DESIGN.md 2>/dev/null || echo "not found"

# 3. tailwind.config.js の確認（v3 の指標）
ls tailwind.config.js 2>/dev/null && echo "v3 config found" || echo "no v3 config"
```

確認結果をもとに以下の決定テーブルに従って進む：

| 条件 | アクション | 次のステップ |
|---|---|---|
| CSS ファイルが 1 つ見つかった | そのファイルを対象として使用する | ステップ 0-B |
| CSS ファイルが複数見つかった | どのファイルに適用するかユーザーに確認する | ステップ 0-B |
| CSS ファイルが見つからない | `@theme` ブロックでフォールバック検索（下記） | ステップ 0-A |
| `tailwind.config.js` が存在する（v3） | 警告メッセージを表示して**停止する** | — |
| `--text-*` 変数が 1 つ以上存在する | 上書き確認をユーザーに求める | ステップ 0-C |
| `--text-*` 変数が存在しない | 確認なしで続行する | ステップ 1 |

### ステップ 0-A: CSS ファイルが見つからない場合のフォールバック

```bash
grep -rl --include="*.css" --exclude-dir=node_modules --exclude-dir=dist '@theme' . 2>/dev/null
```

- 見つかった場合: そのファイルを対象として使用し、ステップ 0-B へ進む。
- それでも見つからない場合: Tailwind CSS エントリーファイルのパスをユーザーに尋ねて**停止する**。

### ステップ 0-B: 確認内容を報告する

対象ファイル内の既存の `--text-*` 変数を確認する：

```bash
grep -n "\-\-text-" {対象ファイル} 2>/dev/null || echo "none"
```

以下のサマリーを表示する：

```
## 現在のプロジェクト状態

対象 CSS:            {path/to/file.css}
DESIGN.md:           あり / なし
tailwind.config.js:  あり（v3） / なし（v4）

タイプスケール（--text-*）:
  見つかりました / 見つかりません
```

**`DESIGN.md` が存在する場合:**
ファイルを読み込んでタイポグラフィ関連の内容を確認し、関連箇所を表示する。

**`tailwind.config.js` が存在する場合（v3）:**
このスキルは Tailwind CSS v4 が必要。警告して**停止する**：

```
⚠️  tailwind.config.js が検出されました — このプロジェクトは Tailwind CSS v3 を使用しているようです。
このスキルは v4（@theme ディレクティブと --text-* 変数）が必要です。
v3 プロジェクトの場合は tailwind.config.js の theme.fontSize でフォントサイズを設定してください。
```

### ステップ 0-C: --text-* 変数が存在する場合の上書き確認

1 つ以上の `--text-*` 変数が見つかった場合、ユーザーに確認する：

- 新しいスケールで上書きする
- キャンセル（既存を維持する）

キャンセルの場合は**停止する**。確認された場合はステップ 1 へ進む。

---

## ステップ 1: パラメーターの収集

ユーザーがプロンプトを呼び出す際に `16px perfect-fourth` のようにテキストを指定した場合、そのテキストをスペース区切りで `[base-size] [scale-name]` としてパースし、Q1/Q2 の確認をスキップしてステップ 2 へ進む。テキストが指定されていない場合、または値が不足している場合は、ユーザーに以下を確認する：

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

各値は **小数点以下 3 桁** に丸める（例: ベース 16px × Perfect Fourth の場合、`text-xs` は `0.563rem`、`text-base` は `1.000rem`、`text-lg` は `1.333rem`）。

> **重要:** `--text-*` 変数に行間を埋め込まない。`--text-base: 1rem 1.5` のような書き方は無効な CSS になるため、rem 値のみを使用すること（`--text-base: 1rem`）。

### 推奨行間（`leading-*` ユーティリティ）

行間は `@theme` 変数ではなく、要素ごとに `leading-*` ユーティリティで制御する：

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

計算結果を表示する：

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

ユーザーに確認する：
- このスケールを適用する（推奨）
- 別のスケールを試す → ステップ 1 に戻る
- キャンセル → 停止する

---

## ステップ 3.5: バックアップ確認

変更を書き込む前に実行する：

```bash
git status --short 2>/dev/null || echo "not a git repo"
```

- **未コミットの変更がある Git リポジトリ:** `{対象ファイル}` が上書きされることを警告する。先に `git stash` または `git commit` を推奨し、ユーザーに確認する（このまま続ける / キャンセル）。
- **クリーンな作業ツリー:** 何も言わずに続行する。
- **Git リポジトリでない場合:** バージョン管理がないことを警告し、手動バックアップを推奨してからユーザーに確認する。

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

### h1〜h6 フォントサイズのマッピング

`@layer base` 内の `h1`〜`h6` に `font-size` 宣言を追加する。`@layer base` ブロックが複数存在する場合は最初のブロックに追記する。既存の `h1`〜`h6` ルールが存在する場合はそのルール内の `font-size` プロパティのみを上書きまたは追加し、他のプロパティは変更しない。

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

---

## ステップ 4.5: h1〜h6 の明示的なテキストサイズクラスの削除

スケールを適用した後、明示的な `text-*` サイズクラスを持つ見出し要素を HTML ファイルからスキャンする（`@layer base` のマッピングを上書きするため）。

### 検出

```bash
grep -rn "<h[1-6][^>]*class=\"[^\"]*text-\(xs\|sm\|base\|lg\|xl\|2xl\|3xl\|4xl\|5xl\|6xl\|7xl\|8xl\|9xl\)" \
  --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.vue" \
  . | grep -v "node_modules" | grep -v "dist"
```

### 候補リスト

一致するものが見つかった場合、リストを表示してからユーザーに確認する：
- 一覧のクラスをすべて削除する（推奨）
- 個別に選択する
- スキップ（すべてそのまま維持する）

> **注意:** `text-ink`、`text-muted`、`text-primary` などのカラークラスは削除しない。削除対象はサイズユーティリティ（`text-xs`〜`text-9xl`）のみ。

---

## ステップ 4.6: DESIGN.md タイポグラフィセクションの更新

`DESIGN.md` 内に `## Typography` セクションが存在し、かつ `fontSize:` キーを含む YAML ブロックがある場合、ユーザーに更新するか確認する。

「更新する」場合：
1. 新しいスケールを使用して関連する見出しレベルの px 値を再計算する
2. `typography:` YAML ブロックの `fontSize` 値を更新する
3. `## Typography` セクションの本文を新しいスケール名と比率に合わせて更新する

---

## ステップ 5: 完了

ロックファイルからパッケージマネージャーを検出する：

```bash
ls pnpm-lock.yaml yarn.lock package-lock.json bun.lockb 2>/dev/null | head -1
```

完了メッセージを表示する：

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

💡 行間はデフォルト値です。要素ごとに leading-* ユーティリティで上書きできます：
   leading-tight（1.25）/ leading-snug（1.375）/ leading-normal（1.5）
   leading-relaxed（1.625）/ leading-loose（2）
```
