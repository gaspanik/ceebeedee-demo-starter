# Figma → Code ワークフロー（逐次サブエージェント）

Figma URL を受け取り、**実装 → レビュー → 最適化** の3フェーズをサブエージェントで逐次実行するワークフロースキルです。

## 引数

`$ARGUMENTS` に Figma URL が渡されます。

## 実行ルール

- 各フェーズは **Agent ツール**でサブエージェントとして起動する
- 必ず **Phase 1 → Phase 2 → Phase 3** の順に実行する（並列不可）
- 各エージェントの完了を確認してから次を起動する
- 全フェーズ完了後、各エージェントの報告内容をまとめてユーザーに提示する

---

## Phase 1: 実装

Agent ツールで以下のプロンプトを渡してサブエージェントを起動する。

```
以下のFigma URLのデザインを実装してください：$ARGUMENTS

## 手順

### 1. デザイン情報の取得
- `get_design_context` でデザインコンテキストとリファレンスコードを取得する
- `get_variable_defs` でデザイントークン（色、スペーシング等）を取得する
- ファイル中のアノテーションの内容を確認し、実装に必要な情報を把握する

### 2. 既存コードの確認
- `src/components/` 内の既存コンポーネントを確認し、再利用できるものがないか調べる
- `src/index.css` の `@theme` ブロックで定義済みのトークンを確認する
- `src/lib/utils.ts` の `cn()` 関数を必ず使用する

### 3. 実装
CLAUDE.md のガイドラインに厳密に従って実装すること：

**コンポーネント配置**
- 新規コンポーネントは `src/components/` に配置
- ファイル名はPascalCase（例: `HeroSection.tsx`）

**スタイリング**
- Tailwind CSS v4 構文を使用（`space-x/y-*` は使わず `gap-*` + flex/grid を使う）
- クラスの合成には必ず `cn()` を使用（`import { cn } from '@/lib/utils'`）
- 複数バリアントがある場合はCVA（`class-variance-authority`）を検討
- 複数の子要素にまたがるスタイルは `tailwind-variants` を検討
- アイコンは `lucide-react` を使用（インライン: `w-4 h-4`、見出し: `w-6 h-6`）

**TypeScript**
- `import React from 'react'` は書かない（react-jsx transform使用）
- フックは named import: `import { useState } from 'react'`
- propsは必ずTypeScriptのinterfaceで明示的に定義する
- `ComponentProps<'button'>` 等を使って既存HTML要素を拡張する

**アクセシビリティ**
- セマンティックなHTMLタグを使用（`header`, `main`, `section`, `nav` 等）
- `aria-label`, `aria-expanded`, `aria-controls` 等の適切なARIA属性を付与
- 画像には意味のある `alt` テキストを設定

**デザイントークン**
- Figmaから取得したトークンが `@theme` に未定義の場合、`src/index.css` に追加する
- 任意値（`w-[42px]`）は最終手段として使用し、複数箇所で使う場合は `@theme` に追加する

**画像アセット**
- Figma MCP の画像URL（`https://www.figma.com/api/mcp/asset/...`）は **7日間で失効** するため、必ずローカルに保存する
- `src/assets/images/` に `webp` 形式でダウンロードし、内容を表す kebab-case で命名する
  ```bash
  curl -s -o src/assets/images/<name>.webp "<figma-asset-url>"
  ```
- コード内では直接URLを埋め込まず、`getImage()` で参照する
  ```tsx
  import { getImage } from '@/lib/image'
  <img src={getImage('hero')} alt="..." />
  ```

### 4. 完了確認
実装完了後、以下を確認してメッセージに含める：
- 作成・変更したファイルの一覧
- 新たに `@theme` に追加したトークン（あれば）
- 既存コンポーネントの再利用状況
- ダウンロードした画像ファイルの一覧（`src/assets/images/` に保存したもの）
- アノテーションで指定された要件をすべて満たしているか
```

---

## Phase 2: レビューと修正

Phase 1 完了後、Agent ツールで以下のプロンプトを渡してサブエージェントを起動する。

```
以下のFigma URLのデザインと実装を比較してレビューしてください：$ARGUMENTS

## 手順

### 1. Figmaのスクリーンショット取得
- `get_screenshot` で対象ノードのスクリーンショットを取得する
- スクリーンショットを参照しながら実装との視覚的差異を特定する

### 2. 視覚的差異の確認と修正
スクリーンショットと実装を比較して以下を確認・修正する：

**レイアウト**
- 要素の配置・整列（flex/grid の方向、justify/align の設定）
- スペーシング（padding, margin, gap の値）
- サイズ（幅・高さ）

**スタイル**
- 色（`@theme` トークンまたはTailwindのカラーパレット）
- タイポグラフィ（フォントサイズ、ウェイト、行間）
- 角丸（border-radius）、シャドウ（box-shadow）
- ボーダー（色・太さ・スタイル）

**レスポンシブ**
- モバイル（`sm:`）・タブレット（`md:`）・デスクトップ（`lg:`）での見た目

### 3. TypeScript エラー・警告の解消
```bash
pnpm build
```
を実行してTypeScriptのエラーと警告をすべて解消する。

よくある問題：
- `noUnusedVariables`: 使っていない変数・インポートの削除
- `noExplicitAny`: `any` 型を具体的な型に置き換える
- `useExhaustiveDependencies`: `useEffect` 等の依存配列の修正

### 4. アクセシビリティの確認
- キーボードナビゲーションが機能するか
- `aria-label` 等のARIA属性が適切に設定されているか
- 色コントラストがWCAG AA基準（4.5:1）を満たしているか
- 画像に意味のある `alt` テキストがあるか

### 5. コードスタイルの確認
```bash
pnpm check
```
を実行してBiomeのリント・フォーマットエラーを自動修正する。

### 6. レビュー結果の報告
修正完了後、以下をまとめて報告する：
- 視覚的差異として修正した箇所
- TypeScriptエラー・警告の修正内容
- アクセシビリティの改善点
- 残課題があれば明記する
```

---

## Phase 3: 最適化

Phase 2 完了後、Agent ツールで以下のプロンプトを渡してサブエージェントを起動する。

```
`src/components/` 内の実装済みコンポーネントをリファクタリングして、再利用性と保守性を高めてください。

## 手順

### 1. 現状の把握
`src/components/` 内のファイルをすべて確認して以下を特定する：
- 重複しているスタイルやロジック
- ハードコードされた値（色、サイズ等）
- より適切なコンポーネントパターンに移行できるもの

### 2. コンポーネントパターンの最適化
CLAUDE.md の「Component Patterns」セクションに従い、各コンポーネントに最適なアプローチを選択する：

**cn 関数** — 条件分岐が少ないシンプルなコンポーネント
**CVA（class-variance-authority）** — バリアントが複数ある単一要素コンポーネント（ボタン、バッジ等）
**tailwind-variants** — 複数の子要素にまたがるバリアントが必要なコンポーネント（カード、フォーム等）

### 3. デザイントークンの整理
- ハードコードされた色・スペーシング値を `src/index.css` の `@theme` ブロックに移動する
- 任意値（`w-[42px]`）を複数箇所で使っている場合はトークン化する
- トークン名はセマンティックに命名する（`--color-brand-primary` 等）

### 4. 共通パターンの抽出
- 3箇所以上で同じスタイルの組み合わせが使われていたら共通コンポーネント化を検討する
- ただし過度な抽象化は避ける（DRYより明快さを優先）

### 5. 型安全性の強化
- `any` 型をすべて排除する
- `ComponentProps<'element'>` を活用してHTML属性を継承する
- `className` propを受け取り `cn()` で外部からのオーバーライドを許容する

### 6. コードスタイルの最終確認
```bash
pnpm check
```
を実行してBiomeのルールに準拠していることを確認する。

### 7. 最適化結果の報告
以下をまとめて報告する：
- リファクタリングしたコンポーネントと変更内容
- `@theme` に追加・整理したトークン
- 新たに作成した共通コンポーネント（あれば）
- 選択したコンポーネントパターンとその理由
```

---

## 最終報告

3フェーズすべて完了後、以下の形式でまとめて報告する：

```
## Figma ワークフロー完了

### Phase 1: 実装
[Agent 1 の報告内容を要約]

### Phase 2: レビュー
[Agent 2 の報告内容を要約]

### Phase 3: 最適化
[Agent 3 の報告内容を要約]
```
