# このリポジトリにおけるこのスキルについて

`publish-to-pages` は、任意のコンテンツ（PPTX、PDF、HTML、Google スライド）を変換し、**新しい** GitHub Pages リポジトリに公開するための汎用スキルです。

**これは、このリポジトリのサイトのデプロイワークフローでは ありません。** このリポジトリのワークショップサイトは、[`.github/workflows/pages.yml`](../../workflows/pages.yml) によってビルド・デプロイされます。そのワークフローは `main` へのプッシュ時に自動で実行され、`website/dist/` 配下の Astro + Starlight のビルドを <https://github-samples.github.io/copilot-workshops/> に公開します。

`publish-to-pages` は、たとえばスライドデッキや単発の HTML アーティファクトなど、*別の* Pages サイトを立ち上げたいときに使ってください。このリポジトリのデプロイには使わないでください。

出典: [`github/awesome-copilot@65b20ad`](https://github.com/github/awesome-copilot/tree/65b20ad912305cb9ac6e3cf2b0e65ea35db2d1f7/skills/publish-to-pages) からインポート。
