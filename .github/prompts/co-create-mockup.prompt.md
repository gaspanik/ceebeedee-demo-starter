---
description: UIモックアップを作成する。「モックアップを作りたい」「新しいページを作って」「UIデザインを実装したい」「LP/トップページを作って」などの場合に使う
tools:
  - search/codebase
  - edit/editFiles
  - execute/getTerminalOutput
  - execute/runInTerminal
---

ユーザーの要件をヒアリングして、プロジェクトの技術スタック（React 19 + TypeScript + Tailwind CSS v4 + TanStack Router）に沿ったUIモックアップを実装する。

> **注意**: ターミナルコマンドを実行する際は `runInTerminal` ツールを使用すること。

## Step 1: サイト・アプリの種類を確認する

まず、何を作りたいか聞く。一問一答で進めると UX がいいので、最初の質問はこれだけでいい:

> 「どんなサイト・アプリの画面を作りたいですか？（例：ECサイト、コーポレートサイト、ダッシュボード、LP、ブログ、SaaSアプリなど）」

ユーザーの回答から用途・ターゲットユーザー・コンテンツの大まかなイメージを把握する。

---

## Step 2: デザインテイストを確認する

次にデザインの方向性を確認する:

> 「デザインのイメージを教えてください:
> - カラー: ブランドカラーや使いたい色はありますか？（なければ「おまかせ」でも可）
> - テイスト: モダン・ミニマル・ナチュラル・ビビッド・クラシックなど、雰囲気のイメージは？
> - 参考サイトや気になるデザインがあれば教えてください（なくてもOK）」

ユーザーが「おまかせ」と言った場合は、Step 1 で聞いたサイト種別に合わせて適切なカラーとテイストを自分で決めて、決定内容をユーザーに一言伝えてから進む。

### カラーを決めたら @theme に定義する

ユーザーまたは自分で決めたカラーは `src/index.css` の `@theme` ブロックに追加する:

```css
@theme {
  --color-primary: #2C4A7C;
  --color-secondary: #F4A261;
  --color-accent: #E76F51;
  --color-base: #FAF7F2;
}
```

Tailwind CSS v4 なので `tailwind.config.js` は使わない。

---

## Step 3: ページ種別と優先順位を確認する

> 「どのページを作りますか？複数ある場合は優先順位も教えてください。
> - トップページ（ヒーロー・特徴・CTA など複数セクション）
> - 商品一覧 / 詳細ページ
> - 記事一覧 / 詳細ページ
> - フォーム（お問い合わせ・サインアップ など）
> - その他（具体的に）」

ページが複数ある場合は1ページずつ実装し、完成したら次を聞くスタイルにする。

---

## Step 4: 実装前のクリーンアップ

`src/routes/__root.tsx` を読み込み、`<Outlet />` の周囲に余分なラッパーや padding があれば除去する:

```tsx
import { createRootRoute, Outlet } from '@tanstack/react-router'
import { TanStackRouterDevtools } from '@tanstack/react-router-devtools'

const RootLayout = () => (
  <>
    <Outlet />
    <TanStackRouterDevtools position="bottom-right" />
  </>
)

export const Route = createRootRoute({ component: RootLayout })
```

ヘッダー・フッターをグローバルに配置する場合はここに追加してよい。ただし `<Outlet />` を直接囲む div に padding/margin を与えないこと。

---

## Step 5: 実装する

### ファイル配置のルール

| 種類 | 配置先 |
|------|--------|
| ページコンポーネント（ルート） | `src/routes/` |
| 再利用可能なUIコンポーネント | `src/components/` |
| カラー・テーマ定義 | `src/index.css` の `@theme` ブロック |

**ルートは `src/routes/` に置くだけで自動登録される**（TanStack Router が `routeTree.gen.ts` を自動生成するため手動編集不要）。

### ルートファイルの命名規則

| URL | ファイルパス |
|-----|------------|
| `/` | `src/routes/index.tsx` |
| `/about` | `src/routes/about.tsx` |
| `/products` | `src/routes/products/index.tsx` |
| `/products/:id` | `src/routes/products/$productId.tsx` |
| `/blog/:slug` | `src/routes/blog/$slug.tsx` |

### コンポーネントの分割方針

できるだけ細かく分割して再利用しやすくする。ルートファイルはページ全体の骨格だけを持ち、実際の UI は全てコンポーネントに切り出す。

分割の目安:
- セクション単位（`HeroSection`, `FeatureSection`, `CtaSection`）
- 繰り返し要素（`ProductCard`, `KpiCard`, `ArticleCard`）
- 独立して機能するUI部品（`FilterBar`, `SortDropdown`, `Pagination`）

関連コンポーネントが複数あるときはサブディレクトリにまとめる（例: `src/components/dashboard/`）。

### コンポーネントのスタイリング

用途に合わせて3つのアプローチを使い分ける:

**シンプルな条件付きスタイル → `cn()` 関数**
```tsx
import { cn } from '@/lib/utils'

function Badge({ active, className }: { active?: boolean; className?: string }) {
  return (
    <span className={cn('px-2 py-1 rounded text-sm', active && 'bg-primary text-white', className)}>
      ...
    </span>
  )
}
```

**単一要素のバリアント → CVA**
```tsx
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const button = cva('px-4 py-2 rounded font-medium transition', {
  variants: {
    intent: { primary: 'bg-primary text-white hover:opacity-90', ghost: 'border border-primary text-primary' },
    size: { sm: 'text-sm', md: 'text-base', lg: 'text-lg px-6 py-3' },
  },
  defaultVariants: { intent: 'primary', size: 'md' },
})
```

**複数要素のバリアント → tailwind-variants**
```tsx
import { tv, type VariantProps } from 'tailwind-variants'

const card = tv({
  slots: { base: 'rounded-xl overflow-hidden', image: 'w-full object-cover', content: 'p-6' },
  variants: { tone: { light: { base: 'bg-white shadow-sm' }, dark: { base: 'bg-gray-900 text-white' } } },
})
```

### 必ず守るルール

- `import React from 'react'` は書かない（react-jsx transform が有効）
- アイコンは `lucide-react` を使う（`import { IconName } from 'lucide-react'`）
- 画像は `src/assets/images/` に置いて `getImage()` で参照（URL を直接埋め込まない）
  - ユーザーが画像を指定しない場合はプレースホルダー（背景色 + テキスト）で代替する
  - 外部URLを指定された場合は curl でローカルにダウンロードしてから参照する:
    ```bash
    curl -L "https://..." -o src/assets/images/hero.jpg
    ```
- `space-x-*` / `space-y-*` は Tailwind v4 では非推奨 → `gap-*` with flex/grid を使う
- レスポンシブ対応必須（`sm:`, `md:`, `lg:` ブレークポイントを活用）
- アクセシビリティ: `<nav aria-label="...">`, `aria-expanded`, `alt` テキストを必ずつける
- ナビゲーションは TanStack Router の `<Link>` コンポーネントを使う
- コンポーネントは named export で書く

---

## Step 6: 実装後の確認

### 1. クラス重複・命名衝突チェック（必須）

各コンポーネントで同じクラスが複数回指定されていないか確認する。

チェック観点:
- **クラスの重複**: 同じ `className` 内に同じクラス名が2回以上ある（例: `text-surface text-surface`）
- **サイズ vs カラーの衝突**: `@theme` で定義した色名と Tailwind 組み込みユーティリティ名が被っていないか（Step 2 の命名規則を参照）
- **意味の競合**: `text-sm` と `text-base` のようにフォントサイズ系クラスが同じ要素に複数ある

### 2. 型チェック・ビルド確認

下記の `<pm>` はお使いのパッケージマネージャ（`npm`・`yarn`・`pnpm` など）に読み替えてください。

```bash
<pm> run check   # Biome リント + フォーマット
<pm> run build   # TypeScript 型チェック + Vite ビルド
```

TypeScript エラーや Vite のビルドエラーがないことを確認する。

### 3. 開発サーバーの起動

ビルドが成功したら、開発サーバーをバックグラウンドで起動する。Vite は他のポートが使用中の場合、5174・5175… と自動でポートを変えるため、**起動後に実際のポートを取得してユーザーに伝える**こと。

手順:

1. `run_in_background: true` でサーバーを起動し、出力ファイルのパスを控える
2. 2〜3秒待ってから出力ファイルを Read する
3. `Local:   http://localhost:XXXX/` の行からポート番号を取得する
4. そのURLをユーザーに案内する

```bash
# 出力ファイルを Read した後、ポートを抽出する例
grep -oE 'localhost:[0-9]+' <出力ファイルパス> | head -1
```

### 4. 報告

ユーザーに完成を報告し、次のアクションを選択してもらう：

```
次に何をしますか？

1. Figma にキャプチャとして取り込む
   → ブラウザのレンダリング結果をそのまま Figma に画像として移植

2. Figma ノードとして生成する
   → コンポーネント・変数・オートレイアウトを持つデザインシステムとして作成

3. Tailwind 変数を Figma Variables に移植する
   → src/index.css の @theme トークンを Figma の Variables として登録

4. このままで終了
```

選択に応じて対応するツール・プロンプトを呼び出す：
- **1 を選択** → 以下の手順で `generate_figma_design` MCP ツールを**直接**呼ぶ：
  1. 開発サーバーの URL（例: `http://localhost:5173`）を使用する
  2. まず `outputMode` なしで `generate_figma_design` を呼び、取り込み先の選択肢を表示する
  3. ユーザーが選択したら `outputMode` と URL を指定して再度呼ぶ
  4. 返却された `captureId` を使い、5秒おきに最大10回ポーリングして完了を待つ
  - **注意**: `use_figma` は呼ばない。純粋なキャプチャのみ。
- **2 を選択** → `use_figma` MCP ツールを直接呼び出して実行する
- **3 を選択** → [co-tailwind-to-figma](co-tailwind-to-figma.prompt.md) プロンプトを起動する
- **4 を選択** → 「お疲れ様でした！」と伝えて終了

---

## よくある落とし穴

- `routeTree.gen.ts` は自動生成されるため **絶対に手動編集しない**
- Tailwind v4 は `@theme` でカスタムトークンを定義（JS設定ファイル不要）
- **`--color-base` は使わない**: `text-base`（font-size: 1rem）と衝突する。`--color-surface` など別名を使う
- パスエイリアスは `@/` で `src/` を指す（`vite.config.ts` と `tsconfig.app.json` 両方で設定済み）
- コンポーネントは named export で書く（default export は TanStack Router のルートコンポーネント以外では使わない）
- **大文字テキスト**: HTML に直接 `ABOUT` と書かず `<span className="uppercase">About</span>` にする（スクリーンリーダー対策）
- **ナビゲーション**: `nav > ul > li > a` の構造を必ず使う。TanStack Router の `<Link>` を `<li>` の中に入れる
