---
name: create-mockup
description: UIモックアップを作成するスキル。ユーザーが「モックアップを作りたい」「新しいページを作って」「UIデザインを実装したい」「サイトの画面を作りたい」「LP/トップページを作って」「デザインを実装して」などと言った場合は必ずこのスキルを使う。サイト種別・デザインテイスト・ページ種別の3つをヒアリングしてから、このプロジェクトのCLAUDE.md仕様に沿って実装する。Figma URLがない状態でUIを作る場合に特に有効。
---

# create-mockup — UIモックアップ作成スキル

ユーザーの要件をヒアリングして、プロジェクトの技術スタック（React 19 + TypeScript + Tailwind CSS v4 + TanStack Router）に沿ったUIモックアップを実装する。

---

## Step 1: サイト・アプリの種類を確認する

まず、何を作りたいか聞く。一問一答で進めると UX がいいので、最初の質問はこれだけでいい:

> 「どんなサイト・アプリの画面を作りたいですか？（例：ECサイト、コーポレートサイト、ダッシュボード、LP、ブログ、SaaSアプリなど）」

ユーザーの回答から用途・ターゲットユーザー・コンテンツの大まかなイメージを把握する。

---

## Step 2: デザインテイストを確認する

次にデザインの方向性を確認する。以下の観点でまとめて質問すると効率的:

> 「デザインのイメージを教えてください:
> - カラー: ブランドカラーや使いたい色はありますか？（なければ「おまかせ」でも可）
> - テイスト: モダン・ミニマル・ナチュラル・ビビッド・クラシックなど、雰囲気のイメージは？
> - 参考サイトや気になるデザインがあれば教えてください（なくてもOK）」

ユーザーが「おまかせ」と言った場合は、Step 1 で聞いたサイト種別に合わせて適切なカラーとテイストを自分で決めて、決定内容をユーザーに一言伝えてから進む。

### カラーを決めたら @theme に定義する

ユーザーまたは自分で決めたカラーは `src/index.css` の `@theme` ブロックに追加する。例:

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

どのページを作るか確認する:

> 「どのページを作りますか？複数ある場合は優先順位も教えてください。
> - トップページ（ヒーロー・特徴・CTA など複数セクション）
> - 商品一覧 / 詳細ページ
> - 記事一覧 / 詳細ページ
> - フォーム（お問い合わせ・サインアップ など）
> - その他（具体的に）」

ページが複数ある場合は1ページずつ実装し、完成したら次を聞くスタイルにする。

---

## Step 4: スターターコンテンツをクリアする

実装に入る前に、スターターのデモコンテンツを必ずクリアする。

### `src/routes/__root.tsx` をクリーンなレイアウトに書き換える

スターターには不要なヘッダーナビが含まれているため、`<Outlet />` のみを返すシンプルな構造に書き換える:

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

### `src/routes/index.tsx` をクリアする

スターターのデモコンテンツを削除し、最小限のプレースホルダーに置き換える（この後すぐ上書きするので内容は何でもよい）。

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

TanStack Router のファイルベースルーティング規則に従う:

| URL | ファイルパス |
|-----|------------|
| `/` | `src/routes/index.tsx` |
| `/about` | `src/routes/about.tsx` |
| `/products` | `src/routes/products/index.tsx` または `src/routes/products.tsx` |
| `/products/:id` | `src/routes/products/$productId.tsx` |
| `/dashboard` | `src/routes/dashboard.tsx` |
| `/blog/:slug` | `src/routes/blog/$slug.tsx` |

**既存ファイルの上書きについて**: このスキルは新規モックアップ作成用なので、`index.tsx` を含む既存ファイルへの上書きは問題ない。

### コンポーネントの分割方針

**できるだけ細かく分割して再利用しやすくする**。ルートファイルはページ全体の骨格だけを持ち、実際の UI は全てコンポーネントに切り出す。

分割の目安:
- セクション単位（`HeroSection`, `FeatureSection`, `CtaSection`）
- 繰り返し要素（`ProductCard`, `KpiCard`, `ArticleCard`）
- 独立して機能するUI部品（`FilterBar`, `SortDropdown`, `Pagination`）
- ナビゲーション・ヘッダー・フッター

**悪い例（避ける）**: `products.tsx` 1ファイルに全JSXを書く  
**良い例**: `products.tsx` が `ProductGrid`, `ProductCard`, `FilterBar` をインポートして組み合わせる

関連コンポーネントが複数あるときはサブディレクトリにまとめる（例: `src/components/dashboard/`）。

### コンポーネントの書き方

プロジェクトには3つのスタイリングアプローチがある。用途に合わせて選ぶ:

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
- 画像は `src/assets/images/` に置いて `getImage()` / `getImageAsync()` で参照
- `space-x-*` / `space-y-*` は Tailwind v4 では非推奨 → `gap-*` with flex/grid を使う
- レスポンシブ対応必須（`sm:`, `md:`, `lg:` ブレークポイントを活用）
- アクセシビリティ: `<nav aria-label="...">`, `aria-expanded`, `alt` テキストを必ずつける
- ナビゲーションは TanStack Router の `<Link>` コンポーネントを使う

### ヒーローセクションのサンプル構造（参考）

```tsx
// src/routes/index.tsx
import { createFileRoute } from '@tanstack/react-router'
import { HeroSection } from '@/components/HeroSection'
import { FeatureSection } from '@/components/FeatureSection'
import { CtaSection } from '@/components/CtaSection'

export const Route = createFileRoute('/')({
  component: HomePage,
})

function HomePage() {
  return (
    <main>
      <HeroSection />
      <FeatureSection />
      <CtaSection />
    </main>
  )
}
```

---

## Step 6: 実装後の確認

実装が終わったら:

1. `<pm> run dev` でdev serverを起動してブラウザで確認
2. `<pm> run check` でBiomeの型チェック・フォーマットを実行
3. レスポンシブ・アクセシビリティに問題がないか確認
4. ユーザーに完成を報告し、次のページ or 修正要望を聞く

---

## よくある落とし穴

- `routeTree.gen.ts` は自動生成されるため **絶対に手動編集しない**
- Tailwind v4 は `@theme` でカスタムトークンを定義（JS設定ファイル不要）
- パスエイリアスは `@/` で `src/` を指す（`vite.config.ts` と `tsconfig.app.json` 両方で設定済み）
- コンポーネントは named export で書く（default export は TanStack Router のルートコンポーネント以外では使わない）
