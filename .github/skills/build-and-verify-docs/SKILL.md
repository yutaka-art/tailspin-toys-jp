---
name: build-and-verify-docs
description: Build, preview, and verify the Copilot Workshops Astro + Starlight workshop site before committing or opening a PR. Use whenever an author or agent is about to build the site, run a local preview/dev server, check links with lychee, confirm the page-count invariant, run the pre-commit verification sequence for any change under `docs/` (content) or `website/` (tooling), or make a PR-time consistency pass to catch structural drift (renamed paths, stale skill/instruction references, inaccurate CI claims, out-of-date structure trees).
---

# ドキュメントサイトのビルドと検証

ワークショップコンテンツは、リポジトリルートの `docs/` ディレクトリにあるプレーンな Markdown です。それを公開する Astro + Starlight サイトは `website/` にあります（ローダーの `base: '../docs'` 経由でコンテンツを取得）。このスキルは、そのサイトを **どうビルドし、プレビューし、検証するか** の単一の情報源です。instruction ファイル（`.github/instructions/*`、`.github/copilot-instructions.md`）は *コンテンツがどうあるべきか* を記述します。このスキルは *ツールをどう実行するか* を記述します。

ステップで別途指定がない限り、すべてのコマンドは **リポジトリルート** から実行してください。

## 使うタイミング

次のときにこのスキルを使います:

- サイトをビルドしようとしている（`npm run build`）、または開発サーバーを起動しようとしているとき。
- ローカルでコンテンツをプレビューする必要があるとき。
- `docs/`（コンテンツ）または `website/`（ツール）配下の変更に対して、コミット前 / PR 前の検証パスを実行しているとき。
- ページ数の不変条件を確認したい、またはリンクをチェックしたいとき。
- `docs/` または `website/` に影響する PR を作成・更新しようとしているとき。

ページが実際にレンダリングされるか（コンソールエラー、壊れた画像、マウントされたコンポーネント）を確認する、任意の **より深いブラウザベース** のパスには、ここの静的チェックの後に [`validate-site-playwright`](../validate-site-playwright/SKILL.md) スキルを使ってください。

## ローカルプレビュー

Astro の開発サーバーが主なプレビュー手段です（ホットリロード）:

```bash
cd website && npm install && npm run dev
```

<http://localhost:4321/copilot-workshops/> を開きます。レッスンコンテンツはリポジトリルートの `docs/` ディレクトリにあります。ローダーが `base: '../docs'` 経由で取得するため、プレビューにシンボリックは不要です。

## 検証シーケンス（コミットのたびに実行）

3 つすべてを実行します。どれかが失敗したらコミットしないでください。

### 1. ビルド（クリーン）

```bash
cd website && rm -rf dist && npm run build
```

### 2. ページ数の不変条件

このワークショップには 36 の異なるルート slug があります。Starlight は各ルートを、英語ルートロケールと、設定された 5 つのローカライズルートについて出力し、翻訳がない場合は英語のフォールバックコンテンツを使います。したがって、ビルド済みサイトには $36 \times 6 = 216$ のワークショップルートに加えて、1 つのレガシーリダイレクト（`/shared/0-prereqs/`。`website/src/pages/shared/0-prereqs.astro` に完全な HTML リダイレクトページとして作成）が含まれます。404 ページを除外した場合の期待される件数は 217 つの `index.html` ファイルです。Astro は 404 ページを含めるため 218 の HTML ファイルと報告します。

```bash
# 英語ルートロケールの異なるルート slug（docs/README.md + docs/<harness>/*.md）
find docs -maxdepth 2 -name '*.md' \
  ! -path 'docs/es-es/*' \
  ! -path 'docs/ja-jp/*' \
  ! -path 'docs/ko-kr/*' \
  ! -path 'docs/pt-br/*' \
  ! -path 'docs/zh-cn/*' | wc -l
# ビルドされたページ（404 を除外）
find website/dist -name index.html | grep -v 404 | wc -l
```

`built pages`（ビルドされたページ）は `(異なるルート slug × 設定されたロケール) + 1` と一致するはずです。ルートやロケールの変更がないのにビルドがそれより **多く** のページを出力する場合は、ローカライズされたコンテンツが余分な親ディレクトリではなく `docs/<locale>/` の直下にあることを確認し、次に `website/src/content.config.ts` のアンダースコアディレクトリ除外をチェックします。この除外は依然として必要で、`_images/` などの補助ディレクトリがページとしてルーティングされないようにします。

### 2b. 翻訳が実際にレンダリングされるか（黙って英語にフォールバックしていないか）

ビルドと上記のページ数は、**実際にどのコンテンツがレンダリングされるかには目が届きません** — ネストが間違っていたり、ロケールツリーの識別が間違っていても、英語フォールバックから配信される 217 ページが出力されます。既知の翻訳ページが翻訳テキストと正しい `lang` 属性を持っていることを確認します:

```bash
grep -o '<title>[^<]*</title>' website/dist/es-es/app/2-add-star-rating/index.html   # スペイン語のタイトル
grep -o 'lang="[^"]*"' website/dist/es-es/app/2-add-star-rating/index.html | head -1  # lang="es-ES"
```

スペイン語のタイトルは英語の文字列ではなく `Lección 2 - Ejecutar tu primera sesión de agente` と表示されるはずです。2 つ目のロケール（例: `ja-jp` -> `lang="ja-JP"`）も抽出チェックします。

### 3. リンクチェック（lychee、オフライン）

サイトは `base=/copilot-workshops/` でビルドされるため、内部 href は絶対パス（`/copilot-workshops/foo/`）です。lychee が内部リンクをたどれるように、そのプレフィックスを `website/dist` にシンボリンクします:

```bash
mkdir -p /tmp/lychee-root && ln -sfn "$PWD/website/dist" /tmp/lychee-root/copilot-workshops \
  && lychee --offline --no-progress --root-dir /tmp/lychee-root 'website/dist/**/*.html'
```

lychee はオフラインで実行され、壊れた **外部** GitHub URL は検出しません。絶対パスの `https://github.com/...` リンクを変更したときは、手動でクリックして確認してください。

## CI が強制すること vs. ローカルのみのこと

`.github/workflows/pages.yml` は PR と `main` へのプッシュで実行されます。実行するのは **次だけ** です:

1. `npm ci`
2. `npm run build`（Astro ビルド）— 成功する必要があります
3. `website/dist/` に対する lychee のオフラインリンクチェック — パスする必要があります

`main` へのプッシュ後、`pages.yml` は `website/dist` を GitHub Pages にデプロイします。ブラウザ検証とコンテンツ整合性分析は、Pages のビルドジョブの一部ではなく、別の任意/セーフティネットのワークフローです。

## PR 時の整合性パス

上記のビルドとリンクチェックは *機械的な* 壊れを検出します。しかし *構造的なずれ* — ファイルが移動したり規約が変わったりしたときに黙って同期が崩れる散文や参照資料 — は検出 **しません**。PR を作成・更新する前に、変更が影響したすべてに対して整合性パスを実施してください:

- **ファイルやフォルダーを名前変更・移動した？** リポジトリ全体を旧パスで grep し、すべてのヒットを更新します — `.md`、instruction ファイル、スキル、そして `README.md`、`docs/README.md`、`website/README.md`、`AUTHORING.md`、`.github/copilot-instructions.md` 内のリポジトリ構造ツリー。例: `images/` が `_images/` になったときは、すべての `../images/...` 参照とすべての構造ツリーを変更する必要がありました。
- **スキル、エージェント、instruction ファイル、ワークフローを追加・削除した？** 旧名への参照を grep して削除します。新しい `.github/instructions/*.instructions.md` ファイルを `AUTHORING.md` の **Deeper conventions** リストに追加し、構造的なものであれば `.github/copilot-instructions.md` の構造ブロックにも追加します。
- **重複したレッスン散文を変更した？** `check-content-alignment` スキルを実行して、同じ更新が必要な他のインラインコピーを特定します。`.github/workflows/content-alignment.md` のエージェント型ワークフローは、セーフティネットとして PR 上で同じ分析を実行します。
- **CI の動作をどこかで説明した？** ビルドと lychee のリンクチェックを実行する `.github/workflows/pages.yml` と一致していることを確認します。
- **ビルドまたは検証のステップを変更した？** このスキルが単一の情報源です。`README.md`、`AUTHORING.md`、`CONTRIBUTING.md` はコマンドを再度文書化せず、*ここを指す* べきです。それらのファイル内の要約はこのスキルと一貫させてください。
- `README.md`、`docs/README.md`、`website/README.md`、`AUTHORING.md`、`.github/copilot-instructions.md` 内の **リポジトリ構造ツリー** は、すべて実際のツリーを反映する必要があります。トップレベルのコンテンツディレクトリを追加・名前変更した場合は、そのすべてを更新します。
- **ページ数の不変条件**（上記セクション 2）は、ビルド後も依然として成り立つはずです。

迷ったときは、`grep -rn "<old-name>" --include='*.md' .`（`node_modules` と `website/dist` を除外）が、古い参照を浮かび上がらせる最も速い方法です。

## クイックリファレンス

```bash
# リポジトリルートから
cd website && rm -rf dist && npm run build && cd ..
mkdir -p /tmp/lychee-root && ln -sfn "$PWD/website/dist" /tmp/lychee-root/copilot-workshops \
  && lychee --offline --no-progress --root-dir /tmp/lychee-root 'website/dist/**/*.html'
```
