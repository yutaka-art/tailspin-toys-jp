# Copilot Workshops — ワークショップコンテンツ

**Copilot Workshops** のワークショップコンテンツです。ソフトウェア開発ライフサイクル全体を通じて、GitHub Copilot のエージェント機能（Copilot CLI、VS Code のエージェントモード、Copilot アプリ、Copilot クラウドエージェント）をガイド付きで学べる教材です。

公開サイトは **<https://github-samples.github.io/copilot-workshops/>** で閲覧できます。

> [!NOTE]
> 受講者がワークショップを通じて構築するデモアプリケーション（Tailspin Toys、純粋な Astro 製のクラウドファンディングサイト。SSR、API エンドポイント、Drizzle データレイヤーを備える）は、別のリポジトリ **<https://github.com/github-samples/tailspin-toys>** で管理されています。このリポジトリが保持するのは*コンテンツ*のみ（レッスンの Markdown、画像、およびそれらを公開する Astro + Starlight サイト）です。

## ワークショップを始める

公開サイトにアクセスしてください: <https://github-samples.github.io/copilot-workshops/>。

## 執筆について

コンテンツを追加・編集したい場合は、まず **[AUTHORING.md](./AUTHORING.md)** をご覧ください。全体像の考え方、ファイル構成、レッスンや画像を追加するための手順が記載されています。

PR / CI のルールについては **[CONTRIBUTING.md](./CONTRIBUTING.md)** を参照してください。

## リポジトリ構成

- **`docs/`** — **レッスンのソース（プレーンな Markdown）。編集はここで行います。** ビルド不要で github.com 上で直接閲覧できます。
  - `README.md` — ワークショップのランディングページ（`slug: index` により公開サイトのホームも兼ねます）。
  - `cli/`、`vscode/`、`cloud/`、`app/` — ハーネスごとのレッスン（Copilot CLI / VS Code / クラウドエージェント / GitHub Copilot アプリ）。codespace ベースの各ハーネスは、それぞれの `0-prerequisites.md` セットアップレッスンで始まり、フォルダーの `README.md`（フォルダーに一致する `slug:` でルーティング）がランディングページになります。
  - `es-es/`、`ja-jp/`、`ko-kr/`、`pt-br/`、`zh-cn/` — 翻訳された各ロケールのツリー（現在は app ハーネス）。
  - `_images/` — スクリーンショットや図（すべてのロケールで共有）。
- **`website/`** — `docs/` を GitHub Pages に公開するためのオプションの Astro + Starlight サイト。レンダリング後のサイトをセルフホストまたはプレビューする場合にのみ必要です。
  - `astro.config.mjs` — サイト URL、ベースパス、`locales` ブロック、サイドバー。
  - `src/content.config.ts` — コンテンツローダー（`base: '../docs'`）。
  - `src/pages/shared/0-prereqs.astro` — 従来の `/shared/0-prereqs/` URL をホームページへ転送する完全な HTML リダイレクト。
- **`AUTHORING.md`** — 執筆者向けの入口（コンテンツの追加・編集手順）。
- **`CONTRIBUTING.md`** — PR フローと CI 要件。
- **`.github/`**
  - `copilot-instructions.md` + `instructions/*.md` — Copilot 向けの執筆ガイダンス。
  - `agents/`、`skills/` — このリポジトリで Copilot が利用できるカスタムエージェントとスキル。
  - `workflows/pages.yml` — `main` へのプッシュ時にサイトをビルド・デプロイします。

## ローカル開発

リポジトリのルートから実行します:

```bash
cd website
npm install
npm run dev
```

サイトは <http://localhost:4321/copilot-workshops/> で起動します。

## 検証

PR を作成する前に、サイトをビルドして一連の検証（クリーンビルド、ページ数チェック、オフラインリンクチェック（lychee））を実行してください。正式なコマンドは **[AUTHORING.md → Building and verifying](./AUTHORING.md#building-and-verifying)** および [`build-and-verify-docs`](./.github/skills/build-and-verify-docs/SKILL.md) スキルに記載されています。CI（`pages.yml`）ではビルドと lychee のリンクチェックが実行されます。

## ライセンス

MIT — [LICENSE](./LICENSE) を参照してください。

## メンテナー

[CODEOWNERS](./.github/CODEOWNERS) を参照してください。

## サポート

現状のまま（as-is）提供されます。ご質問があれば issue を作成してください。
