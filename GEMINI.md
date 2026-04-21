# GEMINI.md - プロジェクトコンテキスト & 開発ガイド

このファイルは、AIエージェントおよびこのプロジェクトで作業する開発者向けの主要なコンテキストとして機能します。

## 1. プロジェクト概要

**React 19**・**TypeScript**・**Vite 8** で構築されたモダンなシングルページアプリケーション（SPA）です。型安全なファイルベースルーティングに **TanStack Router**、スタイリングに **Tailwind CSS v4** を採用しています。コード品質の管理には **Biome** を使用します。

## 2. 技術スタック

| 技術 | バージョン | 用途 |
| :--- | :--- | :--- |
| **React** | 19.2+ | UIライブラリ（`react-jsx` トランスフォーム使用） |
| **TypeScript** | 6.x | 静的型付け |
| **Vite** | 8.x | ビルドツール（`@vitejs/plugin-react` 使用） |
| **TanStack Router** | 1.144+ | 型安全なルーティング |
| **Tailwind CSS** | 4.x | ユーティリティファーストCSS（`@tailwindcss/vite` 経由） |
| **ユーティリティ** | - | `cn`（clsx + tailwind-merge）、`class-variance-authority`、`tailwind-variants` |
| **アイコン** | - | `lucide-react` |
| **Biome** | 2.3+ | リンター＆フォーマッター |

## 3. 主要コマンド

下記コマンドの `<pm>` はお使いのパッケージマネージャ（`npm`・`yarn`・`pnpm` など）に読み替えてください。

**開発**
- `<pm> run dev`：開発サーバーを起動します（http://localhost:5173）
- `<pm> run preview`：本番ビルドをローカルでプレビューします

**ビルド & 品質チェック**
- `<pm> run build`：TypeScript チェック（`tsc`）＋ プロダクションビルド
- `<pm> run check`：Biome でリント＆フォーマット（自動修正）
- `<pm> run lint`：Biome リンターのみ実行（`--write` で自動修正）
- `<pm> run format`：Biome フォーマッターのみ実行（`--write` で自動修正）

**依存関係のインストール**
- `npm install` / `yarn install` / `pnpm install` のいずれかを使用してください

## 4. プロジェクト構成

```
/
├── src/
│   ├── assets/            # 静的アセット
│   │   └── images/        # 最適化された画像処理
│   ├── components/        # 再利用可能な UI コンポーネント
│   │   └── ButtonCn.tsx   # 'cn' ユーティリティを使ったコンポーネント例
│   ├── lib/               # ユーティリティ関数
│   │   ├── image.ts       # 先行読み込み（イーガーローディング）
│   │   ├── imageAsync.ts  # 遅延読み込み（レイジーローディング）
│   │   └── utils.ts       # 'cn'（clsx + tailwind-merge）を含む
│   ├── routes/            # TanStack Router ファイルベースルート
│   │   ├── __root.tsx     # ルートレイアウト
│   │   └── index.tsx      # ホームページ
│   ├── index.css          # グローバルスタイル & Tailwind インポート
│   ├── main.tsx           # エントリーポイント
│   └── routeTree.gen.ts   # 自動生成ファイル - 手動編集禁止
├── biome.json             # リンター/フォーマッター設定
├── tsconfig.app.json      # アプリ用 TypeScript 設定
└── vite.config.ts         # Vite 設定
```

## 5. コーディング規約 & ベストプラクティス

### React
- **インポート**：`React` をインポートしないこと。フックは名前付きインポートで使用（例：`import { useState } from 'react'`）
- **コンポーネント**：関数宣言を使用。Props はインターフェースで明示的に型付け
- **構造**：セマンティックな HTML タグを使用（`header`、`main`、`footer` など）
- **フラグメント**：`<>...</>` 構文を使用

### TypeScript
- **パスエイリアス**：`src/` からのインポートには `@/` を使用
- **厳格モード**：`noImplicitAny` と `strict` が有効
- **モジュール解決**：`moduleResolution: "bundler"`

### コード品質（Biome）
- **フォーマット**：インデント 2 スペース、JSX ダブルクォート、セミコロンなし（必要な場合を除く）、末尾カンマあり
- **ワークフロー**：コミット前に `<pm> run check` を実行すること

### ルーティング（TanStack Router）
- **ファイルベース**：ルートは `src/routes/` 内のファイルで定義
- **自動生成**：`src/routeTree.gen.ts` は自動生成されます。**手動で編集しないこと**
- **作成方法**：`createFileRoute` を使用してルートコンポーネントを定義
- **ナビゲーション**：`Link` コンポーネントを使用
  ```tsx
  import { Link } from '@tanstack/react-router'
  <Link to="/about" className="[&.active]:font-bold">About</Link>
  ```

### Tailwind CSS v4
- **ドキュメント参照**：利用可能な **Tailwind CSS MCP** ツール（例：`search_tailwind_docs`、`get_tailwind_utilities`）を使って最新の v4 ドキュメントを確認・検証すること
- **設定**：`tailwind.config.js` は作成しない。`src/index.css` 内で CSS 変数または `@theme` ディレクティブを使用
- **インポート**：`src/index.css` に `@import "tailwindcss";` と記述
- **クラス名**：v4 構文を使用（例：`space-x-*` の代わりに `gap-*`、`divide-*` は使用しない）
- **ユーティリティ**：
  - クラスのマージには `cn(...)` を使用
  - バリアント API には `class-variance-authority`（CVA）を使用
  - スロットベースの複数要素スタイリングには `tailwind-variants` を使用
- **ベストプラクティス**：
  - 任意の値（`p-[16px]`）より標準スケール（`p-4`）を優先
  - テキストのカラーコントラストは WCAG AA 基準（4.5:1）を満たすこと
  - インタラクティブな要素には `aria-*` 属性を使用

## 6. コンポーネントパターン

### `cn` ユーティリティ
条件付きクラスを持つシンプルなコンポーネントに使用します。
```tsx
import { cn } from '@/lib/utils'
// className={cn('base', active && 'active', className)}
```

### バリアント API（CVA）
複数のバリアントを持つ単一要素コンポーネントに使用します。
```tsx
import { cva } from 'class-variance-authority'
// const buttonVariants = cva(...)
```

### スロットベース（Tailwind Variants）
複数の DOM 要素を持つコンポーネントに使用します。
```tsx
import { tv } from 'tailwind-variants'
// const card = tv({ slots: { ... }, variants: { ... } })
```

## 7. アセット管理

- **配置場所**：`src/assets/images/` に画像を配置
- **先行読み込み**：`import { getImage } from '@/lib/image'`（文字列を返す）
- **遅延読み込み**：`import { getImageAsync } from '@/lib/imageAsync'`（Promise を返す）
- **ギャラリー**：`getAllImages()` または `getAllImagesAsync()` を使用
- **拡張子**：省略可能（自動解決）

## 8. 重要なルール & 注意事項

1. **自動生成ファイル**：`src/routeTree.gen.ts` を変更しないこと
2. **Tailwind 設定**：`tailwind.config.js` を作成しないこと。`src/index.css` を使用すること
3. **クラスのマージ**：`className` プロパティを受け取る際は必ず `cn()` を使用すること
4. **依存関係の追加**：お使いのパッケージマネージャの `add` / `install` コマンドを使用すること（例：`npm install <pkg>`、`yarn add <pkg>`、`pnpm add <pkg>`）