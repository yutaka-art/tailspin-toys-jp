# Website（Astro + Starlight の公開レイヤー）

ワークショップのコンテンツをドキュメントサイトとしてレンダリングし、<https://github-samples.github.io/copilot-workshops/> の GitHub Pages にデプロイする、オプションの公開レイヤーです。

レッスン自体は、リポジトリルートの [`../docs/`](../docs/) ディレクトリにあるプレーンな Markdown です — ビルド不要で github.com 上で直接閲覧できます。この `website/` プロジェクトは、レンダリング後のページサイトをセルフホストまたはプレビューしたい場合にのみ必要です。執筆者向けのガイダンスは [`../AUTHORING.md`](../AUTHORING.md) を参照してください。

## ローカル開発

このディレクトリから実行します:

```sh
npm install
npm run dev      # http://localhost:4321/copilot-workshops/
npm run build    # dist/ に出力
npm run preview  # 本番ビルドをプレビュー
```

## サイト設定

- `astro.config.mjs` — サイト URL、ベースパス、`locales` ブロック、サイドバー（手動で管理）。
- `src/content.config.ts` — コンテンツコレクションのローダー。`base` が `../docs` を指しているため、レッスンの Markdown はリポジトリルートのコンテンツディレクトリから取得されます。アンダースコア接頭辞のディレクトリを除外するので、`_images/` などの補助アセットはページとしてルーティングされません。
- `src/components/` — サイトシェルのコンポーネント（`.astro`）。レッスンコンテンツはここには **存在しません**。
- `src/pages/` — コンテンツコレクション外のスタンドアロンな Astro ルート。現在は `shared/0-prereqs.astro` のみで、従来の `/shared/0-prereqs/` URL をホームページに転送する完全な HTML リダイレクトです（前提条件は現在、ハーネスごとに `/<harness>/0-prerequisites/` にあります）。

ワークショップのホーム（`/`）は `../docs/README.md` から生成されます（その `slug: index` フロントマターがサイトルートにマッピングします）。このため、同じファイルが GitHub のフォルダービューと Starlight のランディングページの両方を兼ねます。

## デプロイ

`main` へのプッシュ時に、[`../.github/workflows/pages.yml`](../.github/workflows/pages.yml) によって GitHub Pages にデプロイされます。
