# React + TypeScript + Vite + TanStack Router + Tailwind CSS for Demonstration

【CeeBeeDee】モダンな技術スタックを使用したReactアプリケーションのテンプレートです。

## 🚀 技術スタック

- **[React 19](https://react.dev/)** - UIライブラリ
- **[TypeScript](https://www.typescriptlang.org/)** - 型安全な開発
- **[Vite 8](https://vite.dev/)** - 高速ビルドツール
- **[@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/tree/main/packages/plugin-react)** - Fast Refresh
- **[TanStack Router](https://tanstack.com/router)** - 型安全なルーティング
- **[Tailwind CSS v4](https://tailwindcss.com/)** - ユーティリティファーストCSS
- **[Biome](https://biomejs.dev/)** - 高速なリンター・フォーマッター

## 📁 プロジェクト構造

```
ts-swc/
├── public/              # 静的アセット
├── src/
│   ├── assets/         # 画像・フォントなど
│   │   └── images/     # 画像ファイル（jpg, png, webp, svg）
│   ├── components/     # 再利用可能なコンポーネント
│   │   └── ButtonCn.tsx
│   │   ├── ButtonCva.tsx    # CVAを使ったバリアントボタン
│   │   └── CardTv.tsx       # tailwind-variantsを使ったカード
│   ├── lib/            # ユーティリティ関数
│   │   ├── image.ts         # 画像アセット管理（eager loading）
│   │   ├── imageAsync.ts    # 画像アセット管理（lazy loading）
│   │   └── utils.ts         # クラス名結合
│   ├── routes/         # TanStack Routerのルート定義
│   │   ├── __root.tsx
│   │   ├── index.tsx
│   │   └── about.tsx
│   ├── index.css       # グローバルスタイル
│   ├── main.tsx        # エントリーポイント
│   └── routeTree.gen.ts # TanStack Router自動生成ファイル
├── biome.json          # Biome設定
├── package.json        # 依存関係
├── tsconfig.json       # TypeScript設定
├── tsconfig.app.json   # アプリ用TypeScript設定
├── tsconfig.node.json  # Node用TypeScript設定
└── vite.config.ts      # Vite設定
```

## 🛠️ セットアップ

### 依存関係のインストール

```bash
npm install   # または yarn install / pnpm install
```

### 開発サーバーの起動

```bash
npm run dev   # または yarn dev / pnpm dev
```

開発サーバーが起動し、通常 http://localhost:5173 でアクセスできます。

## 📝 利用可能なコマンド

以下の `npm run` はお使いのパッケージマネージャ（`yarn`・`pnpm` など）に読み替えてください。

```bash
# 開発サーバー起動
npm run dev

# プロダクションビルド
npm run build

# ビルドしたアプリのプレビュー
npm run preview

# コードのリント
npm run lint

# コードのフォーマット
npm run format

# リント + フォーマット
npm run check
```

## 🔤 標準書体：Gen Interface JP

このテンプレートはデフォルトの書体として **[Gen Interface JP](https://github.com/yamatoiizuka/gen-interface-jp)** を使用しています。

### 書体の変更方法

書体を変更する場合は、以下の2ファイルを編集してください。

**1. `index.html` — フォントの読み込み**

```html
<!-- 現在の設定（Gen Interface JP） -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gen-interface-jp@latest/all.css" />

<!-- ディスプレイ書体を使う場合 -->
<!-- <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gen-interface-jp@latest/display-all.css" /> -->
```

**2. `src/index.css` — デフォルトフォントの適用**

```css
/* Google Fontsを使う場合はファイル冒頭の @import のコメントを外す */
/* @import url("https://fonts.googleapis.com/css2?family=Noto+Sans+JP&display=swap"); */
@import "tailwindcss";

@theme {
  /* 現在の設定（Gen Interface JP） */
  --default-font-family: "Gen Interface JP", sans-serif;

  /* ディスプレイ書体を使う場合 */
  /* --default-font-family: "Gen Interface JP Display", sans-serif; */

  /* 例：Google Fontsに変更する場合 */
  /* --default-font-family: "Noto Sans JP", sans-serif; */
}
```

## 🎨 Tailwind CSS v4

このプロジェクトはTailwind CSS v4を使用しています。設定は [@tailwindcss/vite](https://tailwindcss.com/docs/guides/vite) プラグインを通じて行われます。

## 🧭 TanStack Router

TanStack Routerは自動的にルート定義を生成します。`src/routes/`配下にファイルを追加すると、自動的にルーティングが設定されます。

- `__root.tsx` - ルートレイアウト
- `index.tsx` - ホームページ（`/`）
- `about.tsx` - About ページ（`/about`）

開発時は [TanStack Router DevTools](https://tanstack.com/router/latest/docs/framework/react/devtools) が利用可能です。

## 📦 主要な機能

- **型安全なルーティング** - TanStack Routerによる完全な型推論
- **高速なHMR** - Viteによる高速なFast Refresh
- **自動コード分割** - TanStack Routerの自動コード分割機能
- **最適化された画像管理** - Viteの`import.meta.glob`による効率的なアセット読み込み
- **パスエイリアス** - `@/` で `src/` にアクセス可能
- **Biome統合** - ESLint + Prettierより高速なツールチェーン

## 🖼️ モックアップ作成コマンド

Figma URLがない状態でUIを作りたい場合に使用するスキルです。Claude Code上で以下のように話しかけるだけで起動します。

| トリガー例 | 説明 |
|---|---|
| `/create-mockup` | スキルを明示的に起動する |
| 「モックアップを作りたい」 | 自然言語でも自動的にスキルが起動する |
| 「新しいページを作って」 | 同上 |
| 「LPを作って」 | 同上 |

### ワークフロー

スキルが起動すると、以下の3点をヒアリングしてから実装します：

1. **サイト・アプリの種類** — ECサイト、ダッシュボード、LP、コーポレートサイトなど
2. **デザインテイスト** — カラー・雰囲気・参考サイトなど（「おまかせ」でも可）
3. **ページ種別** — トップページ、一覧ページ、詳細ページなど

プロジェクトの技術スタック（React 19 + TypeScript + Tailwind CSS v4 + TanStack Router）に沿ったコードを自動生成します。

---

## 🎨 Figma連携コマンド

Claude Code上でFigmaデザインをコードに変換するためのスラッシュコマンドです。
事前にFigma MCPサーバーが接続されている必要があります。

| コマンド | 説明 |
|---|---|
| `/figma:setup-env` | スターターのデモコンテンツを削除し、実装を始める前に一度だけ実行する |
| `/figma:implement-figma <URL>` | FigmaのURLを指定してデザインを実装する |
| `/figma:review-figma <URL>` | FigmaのURLを指定して実装とデザインを比較・修正する |
| `/figma:code-optim` | `src/components/` 内の実装済みコンポーネントをリファクタリングする |
| `/figma-workflow <URL>` | 実装・レビュー・最適化を一括実行するオールインワンコマンド |

### 典型的なワークフロー

個別コマンドで実行する場合：

```bash
# 1. 最初に一度だけ実行してデモコンテンツをクリア
/figma:setup-env

# 2. FigmaのURLを指定してデザインを実装
/figma:implement-figma https://www.figma.com/design/...

# 3. 実装とデザインを比較してレビュー・修正
/figma:review-figma https://www.figma.com/design/...

# 4. コンポーネントのリファクタリング
/figma:code-optim
```

`/figma-workflow` を使うと、上記の2〜4を一括実行できます：

```bash
# setup-env 後、1コマンドで実装・レビュー・最適化を実行
/figma-workflow https://www.figma.com/design/...
```

## 🔧 カスタマイズ

### パスエイリアスの追加

[vite.config.ts](vite.config.ts) の `resolve.alias` セクションで追加のエイリアスを定義できます。

### コンポーネントライブラリ

このプロジェクトには、Tailwind CSSを使用したユーティリティが含まれています：

- `class-variance-authority` - バリアント管理
- `clsx` & `tailwind-merge` - クラス名の結合
- `tailwind-variants` - バリアント定義
- `lucide-react` - アイコンライブラリ
