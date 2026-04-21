# ts-swc Copilot 指示書

## プロジェクト概要

React 19 + TypeScript + Vite 8 + Tailwind CSS 4 と TanStack Router を組み合わせた最小構成のスターターテンプレートです。

## 技術スタック

- **フレームワーク**: React 19.2（`react-jsx` トランスフォーム）
- **ビルド**: Vite 8 + `@vitejs/plugin-react`（Babel による Fast Refresh）
- **ルーター**: TanStack Router（ファイルベースルーティング、ルートツリー自動生成）
- **スタイリング**: Tailwind CSS 4（`@tailwindcss/vite` 経由、`@import "tailwindcss"` で適用）
- **ユーティリティ**: `clsx` + `tailwind-merge`（`cn` 関数として統合）、`class-variance-authority`（バリアント API）、`tailwind-variants`（スロットベースの複数要素スタイリング）
- **アイコン**: `lucide-react`
- **リンター/フォーマッター**: Biome 2.3（`biome.json` で厳格設定）
- **パッケージマネージャー**: npm / yarn / pnpm など、お好みのものを使用可

## キー構成

- **エントリー**: `index.html` → `src/main.tsx` → TanStack Router（`src/routes/__root.tsx`）
- **ルート**: `src/routes/`（`src/routeTree.gen.ts` を自動生成）
- **スタイル**: `src/index.css`（`@import "tailwindcss"` のみ、設定ファイル不要）
- **コンポーネント**: `src/components/`
- **ユーティリティ**: `src/lib/utils.ts`（`cn` 関数）、`src/lib/image.ts`（画像処理）
- **パスエイリアス**: `@/` → `src/`（`vite.config.ts` および `tsconfig.app.json` で設定）
- **TypeScript**: プロジェクト参照（`tsconfig.json` が `tsconfig.app.json` と `tsconfig.node.json` を参照）
- **Vite 設定**: `vite.config.ts` に `base: './'`（相対パスデプロイ）

## アーキテクチャ概要

TanStack Router によるファイルベースルーティングを使った React 19 + TypeScript の SPA です。ルーターは `src/routes/` 内のファイルから `src/routeTree.gen.ts` を自動生成します。**このファイルは手動で編集しないこと**。

### 主要な設計判断

- **`@vitejs/plugin-react`**: Babel による Fast Refresh を使う公式 Vite React プラグイン
- **Tailwind CSS v4**: `@tailwindcss/vite` プラグイン使用（PostCSS 不要）。CSS ファイルで `@import "tailwindcss"` と記述
- **Biome（ESLint/Prettier の代替）**: リントとフォーマットを一括管理、より厳格なデフォルト設定

## 重要な開発ワークフロー

### コマンド

下記の `<pm>` はお使いのパッケージマネージャ（`npm`・`yarn`・`pnpm` など）に読み替えてください。

```bash
<pm> run dev      # 開発サーバー起動
<pm> run build    # TypeScript チェック + Vite ビルド
<pm> run check    # Biome リント + フォーマット（自動修正）
```

依存関係のインストール: `npm install` / `yarn install` / `pnpm install`

### ルートの追加
1. TanStack Router の規約に従って `src/routes/` にファイルを作成:
   - `index.tsx` → `/`
   - `about.tsx` → `/about`
   - `posts/$postId.tsx` → `/posts/:postId`
2. `createFileRoute()` のエクスポートパターンを使用（`src/routes/index.tsx` 参照）
3. ファイル変更時にルーターが自動再生成

### 変更の確認
- 開発サーバーは http://localhost:5173 で起動（3000 ではない）
- 開発モードでは右下に TanStack Router DevTools が表示される

## コーディング規約

### React

- **React インポート不要**: プロジェクトは `react-jsx` トランスフォームを使用 — `import React from 'react'` は**絶対に書かない**
- **フックのインポート**: 名前付きインポートで使用: `import { useState, useEffect } from 'react'`
- **コンポーネント構造**:
  - セマンティックな HTML タグを使用（`header`、`main`、`footer`、`section`、`article`、`nav`、`aside`）
  - Props は TypeScript インターフェースで明示的に型付け
  - 例:
    ```tsx
    interface ButtonProps {
      label: string
      onClick: () => void
    }

    function Button({ label, onClick }: ButtonProps) {
      return <button onClick={onClick}>{label}</button>
    }
    ```

### TypeScript

- **厳格モード**: `strict: true`、未使用変数/パラメータはエラー
- **モジュール**: `moduleResolution: "bundler"`、`allowImportingTsExtensions: true`
- **JSX**: `react-jsx` トランスフォーム（`import React` 不要）
- **パスマッピング**: `baseUrl: "./src"` と `@/*` エイリアスで `src/` からの絶対インポート

### Biome ルール

- **フォーマット**: 2 スペース、LF、80 文字幅、セミコロンは必要に応じて、末尾カンマあり
- **リンター**: `recommended` ベース + 厳格な TypeScript ルール（`noExplicitAny: error`、`noUnusedVariables: error`）
- **React**: `useExhaustiveDependencies: warn`、`useHookAtTopLevel: error`
- コミット前に `<pm> run check` を実行すること。Biome はほとんどの問題を自動修正します。

## Tailwind CSS v4 ガイドライン

### 基本方針

- **設定ファイル不要**: `tailwind.config.js` は作成しない。`src/index.css` に `@import "tailwindcss"` と記述するだけ
- **Vite プラグイン**: `@tailwindcss/vite` が必要（PostCSS 不要）
- **カスタマイズ**: CSS 変数または `@theme` ディレクティブを使用（従来の JS 設定は使わない）

### MCP ツール（利用可能な場合）

Tailwind CSS MCP ツールが環境に存在する場合は活用して、最新ドキュメントとユーティリティを参照してください:

- **ドキュメント検索**: `mcp_tailwindcss_m_search_tailwind_docs` — 特定のトピック・ユーティリティ・概念を検索
- **ユーティリティ取得**: `mcp_tailwindcss_m_get_tailwind_utilities` — カテゴリ・プロパティ・検索ワードで取得
- **設定ガイド**: `mcp_tailwindcss_m_get_tailwind_config_guide` — フレームワーク固有の設定手順
- **コンポーネントテンプレート**: `mcp_tailwindcss_m_generate_component_template` — Tailwind クラス付き HTML テンプレートを生成
- **CSS 変換**: `mcp_tailwindcss_m_convert_css_to_tailwind` — 既存 CSS を Tailwind ユーティリティに変換
- **カラーパレット**: `mcp_tailwindcss_m_get_tailwind_colors` — カラーパレット情報を取得

これらのツールは公式 Tailwind CSS ドキュメントから常に最新情報を提供し、v4 固有の機能・変更点の正確な把握に役立ちます。

### ナビゲーション構造

- **TanStack Router の Link**: ナビゲーションには TanStack Router の `Link` コンポーネントを使用:
  ```tsx
  import { Link } from '@tanstack/react-router'

  <nav aria-label="Main navigation">
    <ul>
      <li><Link to="/">Home</Link></li>
      <li><Link to="/about">About</Link></li>
    </ul>
  </nav>
  ```
- **アクティブ状態のスタイリング**: TanStack Router の `Link` はアクティブリンクに `.active` クラスを自動付与:
  ```tsx
  <Link
    to="/about"
    className="[&.active]:font-bold [&.active]:text-blue-600"
  >
    About
  </Link>
  ```
- **ARIA ラベル**: `<nav>` 要素には説明的な `aria-label` を付与（例: `"Main navigation"`、`"Footer links"`）

### インタラクティブ要素

- **モバイルメニューボタン**: 適切な ARIA 属性を使用:
  ```tsx
  <button
    aria-expanded={isOpen}
    aria-controls="mobile-menu"
    aria-label="Toggle menu"
  >
    Menu
  </button>
  ```
- **状態の明示**: 折りたたみセクションには `aria-expanded` を `true`/`false` で設定
- **コントロールの関連付け**: `aria-controls` でボタンと対象要素をリンク
- **フォーカス管理**: すべてのインタラクティブ要素でキーボードナビゲーションが動作することを確認

### アクセシビリティのベストプラクティス

- **alt テキスト**: 画像には必ず意味のある `alt` 属性を付与
- **カラーコントラスト**: テキストは WCAG AA 基準（通常テキスト 4.5:1）を満たすこと
- **キーボードナビゲーション**: すべての機能をキーボードで操作できること
- **スクリーンリーダーテスト**: macOS は VoiceOver、Windows は NVDA/JAWS でテスト

### テーマ管理

- **@theme ブロックを使用**: `src/index.css` でプロジェクト固有のデザイントークンを定義:
  ```css
  @import "tailwindcss";

  @theme {
    --color-primary: #294779;
    --color-secondary: #f59e0b;
  }
  ```
- **カスタムクラスを参照**: 任意の値（`text-[#294779]`）ではなくテーマ変数（`text-primary`）を使用
- **ハードコードを避ける**: 色・スペーシングは `@theme` に集約して一貫性を保つ

### スペーシング & 値のガイドライン

- **標準スケールを優先**: Tailwind のスペーシングスケール（1単位 = 4px）を基本として使用:
  - ✅ 推奨: `gap-2`（8px）、`p-4`（16px）、`m-6`（24px）、`w-80`（320px）
  - ❌ 非推奨: `gap-[8px]`、`p-[16px]`、`w-[320px]`
- **任意の値は最終手段**: 標準スケールやテーマ変数で実現できない場合のみ `[...]` 構文を使用:
  - 許容: `w-[42px]`（デザイン上ちょうど 42px が必要な場合）
  - より良い: 複数箇所で使う場合は `@theme` に追加
- **レスポンシブデザイン**: 標準ブレークポイントを使用（`sm:`、`md:`、`lg:`、`xl:`、`2xl:`）

### v4 クラス名変更（重要）

Tailwind CSS v4 ではクラス命名規則が変更されています。**常に v4 構文を使用すること**:

```tsx
// ❌ 誤り（v3 構文）
<div className="text-gray-500 bg-gray-50 space-y-4">

// ✅ 正しい（v4 構文）
<div className="text-gray-500 bg-gray-50 flex flex-col gap-4">
```

**主な v4 変更点:**
- ❌ `space-x-*` / `space-y-*` → ✅ flex/grid と組み合わせて `gap-*` を使用
- ❌ `divide-*` → ✅ 各子要素にボーダーを設定
- ✅ 以下のユーティリティはそのまま使用可: `flex`、`grid`、`p-*`、`m-*`、`w-*`、`h-*` など
- ✅ 任意の値: 控えめに使用（`w-[42px]`）。標準スケール（`w-2.5`）や CSS 変数（`text-primary`）を優先
- ✅ レスポンシブ: `sm:`、`md:`、`lg:`、`xl:`、`2xl:`
- ✅ 状態バリアント: `hover:`、`focus:`、`active:`、`disabled:` など

**v3 固有のユーティリティは生成・提案しないこと。** リファクタリング時は非推奨のユーティリティをモダンな flex/grid パターンに変換すること。

### スタイリングガイドライン

- **Tailwind ファースト**: CSS Modules や styled-components は使わない
- **インラインクラス**: コンポーネントのスタイルは `className` プロパティで管理
- **`cn()` で合成**: Tailwind クラスのマージ・重複排除（`clsx` + `tailwind-merge` を使用）
- **アイコン**: `lucide-react` を使用し、サイズを統一（インライン: `w-4 h-4`、見出し: `w-6 h-6`）
- **コンポーネントのオーバーライド**: `className` プロパティを受け取り、`cn` の最後に適用

## コンポーネントパターン

### cn 関数

`cn` ユーティリティ（`src/lib/utils.ts`）は `clsx` と `tailwind-merge` を組み合わせ、条件付きクラス名の処理と Tailwind の競合解決を行います:

```tsx
import { cn } from '@/lib/utils'

// 条件付きクラスの基本的な使用例
<button
  className={cn(
    'base-styles',
    isActive && 'active-styles',
    disabled && 'disabled-styles',
    className  // 外部からのオーバーライドを許可
  )}
>
```

### ボタンコンポーネントのパターン

バリアントスタイルを持つコンポーネントを構築する3つのアプローチ:

**1. シンプルな条件分岐（`ButtonCn.tsx`）**:
- 条件付きスタイリングに `cn` 関数を使用
- バリエーションが少ない、または単一の DOM 要素を持つシンプルなコンポーネントに最適
- 例:
  ```tsx
  import { cn } from '@/lib/utils'
  import type { ComponentProps } from 'react'

  type ButtonProps = ComponentProps<'button'> & {
    active?: boolean
  }

  export const Button = ({ className, active, disabled, ...props }: ButtonProps) => (
    <button
      className={cn(
        'base-classes',
        active && 'active-classes',
        disabled && 'disabled-classes',
        className
      )}
      {...props}
    />
  )
  ```

**2. バリアント API（CVA）**:
- 複雑なバリアントには `class-variance-authority`（CVA）を使用
- 複数のデザインシステムバリアントを持つ単一要素コンポーネントに最適
- `VariantProps<typeof variants>` で型安全なバリアント Props
- 例:
  ```tsx
  import { cva, type VariantProps } from 'class-variance-authority'
  import { cn } from '@/lib/utils'

  const buttonVariants = cva('base-classes', {
    variants: {
      intent: {
        primary: 'primary-classes',
        secondary: 'secondary-classes'
      },
      size: {
        sm: 'small-classes',
        md: 'medium-classes'
      }
    },
    defaultVariants: { intent: 'primary', size: 'md' }
  })

  type ButtonProps = ComponentProps<'button'> & VariantProps<typeof buttonVariants>

  export const ButtonCva = ({ intent, size, className, ...props }: ButtonProps) => (
    <button className={cn(buttonVariants({ intent, size }), className )} {...props} />
  )
  ```

**3. スロットベース（Tailwind Variants）**:
- 複雑な複数要素コンポーネントには `tailwind-variants` を使用
- バリアントが複数の子要素に影響するコンポーネントに最適（カード、フォーム、ナビゲーションなど）
- `twMerge` が組み込み済み（`cn` でラップ不要）
- `VariantProps<typeof config>` で型安全
- 例:
  ```tsx
  import { tv, type VariantProps } from 'tailwind-variants'
  import type { ReactNode } from 'react'

  // スロット付きコンポーネントバリアントを定義
  const card = tv({
    slots: {
      base: 'rounded-lg overflow-hidden shadow-md',
      image: 'w-full h-48 object-cover',
      content: 'p-6',
      title: 'text-xl font-bold mb-2',
      description: 'text-sm mt-2',
    },
    variants: {
      tone: {
        default: {
          base: 'bg-white',
          title: 'text-gray-900',
          description: 'text-gray-500',
        },
        dark: {
          base: 'bg-slate-900',
          title: 'text-white',
          description: 'text-slate-400',
        },
      },
    },
    defaultVariants: {
      tone: 'default',
    },
  })

  // 型安全な Props を抽出
  type CardVariants = VariantProps<typeof card>

  interface CardProps extends CardVariants {
    title: string
    imageUrl?: string
    children: ReactNode
    className?: string
  }

  // コンポーネント実装
  export const Card = ({ tone, title, imageUrl, children, className }: CardProps) => {
    const { base, image, content, title: titleClass, description } = card({ tone })
    
    return (
      <div className={base({ class: className })}>
        {imageUrl && <img src={imageUrl} alt="Thumbnail" className={image()} />}
        <div className={content()}>
          <h3 className={titleClass()}>{title}</h3>
          <div className={description()}>{children}</div>
        </div>
      </div>
    )
  }
  ```

**使い分けの目安:**
- **cn 関数**: シンプルなコンポーネント、条件が少ない、単一要素
- **CVA**: 複数のバリアントの組み合わせを持つ単一要素コンポーネント（ボタン、バッジ）
- **tailwind-variants**: バリアントが複数の子要素に影響する複数要素コンポーネント（カード、フォーム、ナビゲーション）

### アイコン

```tsx
// lucide-react の使用例（src/routes/__root.tsx、src/routes/index.tsx 参照）
import { IconName } from 'lucide-react'

function Component() {
  return <IconName className="w-4 h-4" />
}
```

## アセット管理

### 画像ユーティリティ

プロジェクトは Vite の `import.meta.glob` で最適化された画像処理を行います。画像はすべて `src/assets/images/` に配置してください。

**ファイル**:
- `src/lib/image.ts` - 先行読み込み（同期関数）
- `src/lib/imageAsync.ts` - 遅延読み込み（非同期関数）

**対応フォーマット**: `jpg`、`jpeg`、`png`、`webp`、`svg`

#### getImage()

拡張子あり・なしで画像を取得。省略時は自動解決します。

```tsx
import { getImage } from '@/lib/image'

function Component() {
  return (
    <img 
      src={getImage('portrait.jpg')}  // 拡張子あり
      alt="Portrait" 
    />
    // または
    <img 
      src={getImage('portrait')}      // portrait.jpg を自動検出
      alt="Portrait" 
    />
  )
}
```

- 画像が見つからない場合は空文字列を返す
- 開発モード: 見つからない場合はコンソール警告を出力
- 先行読み込み（ビルド時にすべての画像を読み込み）

#### getImageAsync()

大きな画像をパフォーマンス向上のために遅延読み込み:

```tsx
import { getImageAsync } from '@/lib/imageAsync'
import { useState, useEffect } from 'react'

function Gallery() {
  const [imageUrl, setImageUrl] = useState('')
  
  useEffect(() => {
    getImageAsync('large-photo.jpg').then(setImageUrl)
  }, [])
  
  return imageUrl ? <img src={imageUrl} alt="Large" /> : <p>Loading...</p>
}
```

#### getAllImages()

すべての画像をキーバリューマップで取得（ギャラリーに便利）:

```tsx
import { getAllImages } from '@/lib/image'

function ImageGallery() {
  const images = getAllImages() // { filename: url, ... }
  
  return (
    <div className="grid grid-cols-3 gap-4">
      {Object.entries(images).map(([name, url]) => (
        <img key={name} src={url} alt={name} />
      ))}
    </div>
  )
}
```

#### getAllImagesAsync()

すべての画像を非同期でキーバリューマップとして取得:

```tsx
import { getAllImagesAsync } from '@/lib/imageAsync'
import { useState, useEffect } from 'react'

function ImageGallery() {
  const [images, setImages] = useState<Record<string, string>>({})
  
  useEffect(() => {
    getAllImagesAsync().then(setImages)
  }, [])
  
  return (
    <div className="grid grid-cols-3 gap-4">
      {Object.entries(images).map(([name, url]) => (
        <img key={name} src={url} alt={name} />
      ))}
    </div>
  )
}
```

**ベストプラクティス:**
- 画像はすべて `src/assets/images/` に配置
- 意味のあるファイル名を使用（例: `hero-banner.jpg`、`img1.jpg` は避ける）
- 静的アセットには `getImage()` を優先（先行読み込み、初期表示が速い）
- 大きな画像や条件付き読み込みには `getImageAsync()` を使用
- 非同期関数は `@/lib/imageAsync` からインポート
- 同期関数は `@/lib/image` からインポート
- アクセシビリティのために常に意味のある `alt` テキストを付与

## トラブルシューティング

- **Biome LSP エラー**: VSCode をリロード（`Developer: Reload Window`）、ワークスペースの信頼を確認
- **HMR 停止**: 開発サーバーを再起動、`node_modules/.vite` キャッシュを削除
- **型エラー**: `<pm> run build`（`tsc -b` → `vite build`）で事前確認
- **ルーターが更新されない**: `routeTree.gen.ts` が再生成されているかターミナルのログを確認

## 機能追加時の手順

- **新しいコンポーネント**: `src/components/` に作成、`cn()` を使用、名前付きエクスポート
- **新しいルート**: `src/routes/` に追加、`createFileRoute()` を使用（ルーター設定の手動変更は不要）
- **新しいユーティリティ**: `src/lib/` に追加、純粋な関数で保持、名前付きエクスポート
- **依存関係の追加**: お使いのパッケージマネージャを使用（例: `npm install <pkg>`、`yarn add <pkg>`、`pnpm add <pkg>`）
- **更新**: お使いのパッケージマネージャの update コマンドを使用
```

### パスエイリアス

`src/` からのインポートには `@/` を使用:
```tsx
import { Button } from '@/components/ButtonCn'
import { cn } from '@/lib/utils'
```

## インテグレーションポイント

### ルーター設定（`src/main.tsx`）
- `createRouter({ routeTree })` でルーターインスタンスを作成
- `basepath` にはデプロイ先の柔軟性のために `import.meta.env.BASE_URL` を使用
- TypeScript モジュール拡張でルーターの型安全を確保

### Tailwind 設定
- **v4 構文**: `tailwind.config.js` は不要 — CSS ベースの設定を使用
- Vite 経由でプラグインを読み込み（PostCSS ではない）
- グローバルスタイルは `src/index.css`（`@import "tailwindcss"` の1行のみ）

### 型安全
- `tsconfig.json` はプロジェクト参照を使用（`tsconfig.app.json`、`tsconfig.node.json`）
- パスマッピング: `@/*` → `./src/*`（`vite.config.ts` と `tsconfig.app.json` で同期）

## よくある落とし穴

1. **`routeTree.gen.ts` を編集しない** — `@tanstack/router-plugin` が自動生成するファイル
2. **`tailwind.config.js` を作成しない** — Tailwind v4 は CSS ベースの設定を使用
3. **`cn()` ユーティリティをスキップしない** — className を直接連結すると Tailwind の優先順位が壊れる
4. **Biome は `routeTree.gen.ts` を無視** — `biome.json` の includes/excludes を参照

## 機能追加時の手順

- **新しいコンポーネント**: `src/components/` に作成、`cn()` を使用、名前付きエクスポート
- **新しいルート**: `src/routes/` に追加、`createFileRoute()` を使用（ルーター設定の手動変更は不要）
- **新しいユーティリティ**: `src/lib/` に追加、純粋な関数で保持、名前付きエクスポート
- **依存関係の追加**: お使いのパッケージマネージャを使用（例: `npm install <pkg>`、`yarn add <pkg>`、`pnpm add <pkg>`）
