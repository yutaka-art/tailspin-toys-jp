---
description: 'Astro + Starlight site wrapper conventions'
applyTo: 'website/**/*.{astro,mjs,ts,js}'
---

# Astro + Starlight ラッパー

`website/` は、ワークショップを GitHub Pages に公開する Astro + Starlight プロジェクトです。これはアプリケーションでは **なく**、薄いサイトシェルです。レッスンコンテンツはリポジトリルートの `docs/` ディレクトリにあります（ローダーの `base: '../docs'` 経由で取得）。執筆は `website/` 配下のプロジェクトファイルではなく、そこで行ってください。

## サイト設定

- ベースパス: `/copilot-workshops`（リポジトリの GitHub Pages スラッグ）。
- サイト URL: `https://github-samples.github.io/copilot-workshops/`。
- **サイドバー: 手動で管理** します（`astro.config.mjs` 内）。`sidebar` 配列が、受講者が見る順序とナビゲーションに表示されるページの両方を決めます。新しいレッスンは明示的に追加する必要があります。
- **コンテンツコレクション** は、`src/content.config.ts` 内のカスタム `glob()` ローダー（`base: '../docs'`）を通じて、リポジトリルートの `docs/` ディレクトリから取得されます。このローダーはアンダースコア接頭のファイルとディレクトリを除外するため、`_images/` などの補助アセットがページとしてルーティングされません。フォルダーのランディングページは、Starlight のデフォルトの `index.md` ではなく `README.md` ファイルです（github.com 上でレンダリングされるように）。各ファイルはフロントマターに `slug:` を持ち、インデックスファイルから得られるはずのルートを再現します — `docs/README.md` → `slug: index`（サイトホーム `/`）、`docs/<harness>/README.md` → `slug: <harness>`、ローカライズされたランディングはロケールを接頭辞に付けた slug を使います（`docs/<locale>/README.md` → `slug: <locale>`、`docs/<locale>/<harness>/README.md` → `slug: <locale>/<harness>`）。

## アプリ風のコンポーネントを追加しない

これはドキュメントのラッパーです。インタラクティブなフレームワークアイランド（Svelte、React など）、Tailwind のユーティリティクラスによるスタイリング層、カスタムルーティング、その他アプリケーション風のコードを追加しないでください。Starlight のデフォルトを超えるものはすべて正当化が必要です。

## ビルドと検証

`astro.config.mjs` や `website/src/` 配下のものを変更したら、[`build-and-verify-docs`](../skills/build-and-verify-docs/SKILL.md) スキルでサイトをビルドして検証してください。そのページ数の不変条件は、予期しないルーティングページを検知するトリップワイヤーです: Starlight は 36 のワークショップルートをルート言語と設定された 5 つのロケールについて出力し、さらにレガシーリダイレクトを追加するため、404 ページを除いて 217 個のビルド済み `index.html` ページになります。ルートやロケールの変更がないのに件数が変わった場合は、`docs/` 配下のロケールレイアウトと、`src/content.config.ts` のアンダースコアディレクトリ除外を確認してください。
