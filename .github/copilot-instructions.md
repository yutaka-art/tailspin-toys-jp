# Copilot Workshops — ワークショップ執筆ガイド

このリポジトリは、Astro + Starlight サイトとして <https://github-samples.github.io/copilot-workshops/> で公開される **Copilot Workshops** の **ワークショップコンテンツ** をホストしています。受講者がワークショップを通じて構築するデモアプリケーションは、別のリポジトリにあります: <https://github.com/github-samples/tailspin-toys>。

**これはコンテンツ専用のリポジトリです。** デモアプリのアプリケーションコード（Astro の SSR エンドポイント、Drizzle データレイヤー、UI コンポーネント、Tailwind のスタイル、テスト）をここに追加しないでください。アプリケーションの変更は `tailspin-toys` に属します。

## リポジトリ構成

- `docs/` — **すべてのレッスンのソース Markdown。ここを編集します。** ビルド不要で github.com 上で直接閲覧できます。
  - `README.md` — ワークショップのランディングページ（`slug: index` フロントマターでサイトのホームも兼ねる）。
  - `cli/`、`vscode/`、`cloud/`、`app/` — ハーネスごとのレッスン（Copilot CLI / VS Code / クラウドエージェント / GitHub Copilot アプリ）。各フォルダーのランディングページは `README.md` です（フォルダーパスに一致する `slug:` でルーティング）。各ハーネスはそれぞれの `0-prerequisites.md` セットアップレッスンで始まります。CLI と VS Code のハーネスは codespace をセットアップし、app と cloud のハーネスはそのフローに必要なセットアップをカバーします（app の場合は Node.js のローカルインストールと、テンプレートからのプロジェクト作成）。
  - `es-es/`、`ja-jp/`、`ko-kr/`、`pt-br/`、`zh-cn/` — Starlight が要求するロケールルートパスにあるローカライズされたコンテンツ。翻訳されたページは各ロケールディレクトリ配下で英語のパスをミラーし、未翻訳のページは Starlight の英語フォールバックを使います。
  - `_images/` — スクリーンショットと図（すべてのロケールで共有）。
- `website/` — `docs/` を GitHub Pages に公開するオプションの Astro + Starlight サイト（ローダー `base: '../docs'`）。レンダリング後のサイトをセルフホストまたはプレビューする場合にのみ必要です。
  - `astro.config.mjs` — 手動で管理されるサイドバーと `locales` ブロックを含むサイト設定。従来の `/shared/0-prereqs/` → ホーム（`/`）へのリダイレクトは、`src/pages/shared/0-prereqs.astro` にある完全な HTML リダイレクトページです（`astro.config.mjs` の `redirects` エントリではない。それだと `<html>` 要素のないスタブが出力され、Pagefind がインデックスできないため）。前提条件は現在、ハーネスごと（`/<harness>/0-prerequisites/`）にあるので、古い shared-prereqs URL はホームページに転送されます。
  - `src/content.config.ts` — アンダースコア接頭辞の補助ディレクトリを除外するカスタムコンテンツローダー（`base: '../docs'`）。これにより `_images/` はコンテンツとしてルーティングされません。
- `AUTHORING.md` — 執筆者向けの入口（レッスンや画像を追加するためのレシピ）。
- `CONTRIBUTING.md` — AUTHORING.md への短いポインターと PR/CI ルール。
- `.github/`
  - `copilot-instructions.md` — このファイル。
  - `instructions/` — スコープ付きの instruction ファイル（`applyTo` フロントマターが特定のファイルグロブを対象にする）。
  - `agents/` — Copilot が利用できるカスタムエージェント。
  - `skills/` — Copilot が利用できるスキル（各スキルの役割の索引は [`skills/README.md`](skills/README.md) を参照）。
  - `workflows/pages.yml` — サイトをビルドしてデプロイします。
  - `workflows/content-alignment.md` — 整合した更新が必要な重複コンテンツがないか PR をチェックするエージェント型ワークフロー。

## 執筆規約

### パスをまたいだ散文の再利用

同じ散文が複数のハーネス（CLI、VS Code、cloud）に当てはまる場合は、ハーネスごとの各 `.md` レッスンにインラインでコピーします。インポートベースの共有コンテンツの仕組みはありません。ホストページがフロントマター、見出し、ナビゲーション、本文の散文を所有します。

インラインのコピーはずれていく可能性があるため、重複するセクションを編集した後は `check-content-alignment` スキルを実行してください。`.github/workflows/content-alignment.md` のエージェント型ワークフローは、セーフティネットとして PR 上で同じ分析を実行しますが、影響を受けるすべてのレッスンを更新する代わりとして依存しないでください。

### admonition（コールアウト）

- **どこでも GitHub の admonition 構文を使います** — 公開レッスン *と* リポジトリの Markdown の両方です。`[!NOTE]` / `[!TIP]` / `[!IMPORTANT]` / `[!WARNING]` / `[!CAUTION]` のマーカーを独立した `>` 接頭の行に置き、本文は続く `>` 接頭の行に書きます。
- `docs/**` 配下の公開レッスンでは、`remark-github-admonitions-to-directives` プラグイン（`website/astro.config.mjs` で組み込み）がビルド時にこれらを Starlight のアサイドに変換します（NOTE/IMPORTANT → note、TIP → tip、WARNING/CAUTION → caution）。Starlight の `:::` ディレクティブを書か **ない** でください。
- GitHub 構文にはカスタムタイトルやネストの形式がないため、コールアウトの見出しは **太字の導入行**（`> **Title**` の後に空の `>` 行、その後に本文）に置き、「ネストされた」コールアウトは空行で区切った兄弟のブロッククォートとして出力します。完全な対応関係とパターンは [`.github/instructions/markdown.instructions.md`](instructions/markdown.instructions.md) にあります。

### リンク

- **ワークショップ内:** Markdown の参照スタイルリンク（`[Exercise 1][exercise-1]` と、ページ下部に定義した `[exercise-1]: ../1-foo/`）。
- **外部ドキュメント:** `docs.github.com` やその他の信頼できる情報源への完全な URL。
- **リポジトリ間（テンプレートリポジトリ、サンプルコード）:** `github.com/github-samples/tailspin-toys/...` への完全な URL。*この* リポジトリ内のファイルを、あたかもテンプレートであるかのようにリンクし **ない** でください — `tailspin-toys` が受講者向けのテンプレートです。

## ビルド、プレビュー、検証

サイトのビルド・プレビュー・検証のためのツール（開発サーバー、クリーンビルド、ページ数の不変条件、lychee のリンクチェック）は、[`build-and-verify-docs`](skills/build-and-verify-docs/SKILL.md) スキルにあります。コミットのたびにその検証手順を実行し、いずれかのステップが失敗したらコミットしないでください。

PR を作成または更新する前には、そのスキルに記載された **PR 時の整合性チェック** も実施してください — これは、ビルドやリンクチェックでは検出できない構造的なずれの点検（名前変更されたパス、古くなったスキル / instruction への参照、CI の記述、リポジトリ構造ツリー、コピーした散文の整合性）です。

## コミットの衛生

- Conventional Commits のプレフィックスを推奨します（`docs:`、`chore:`、`fix:`）。
- 常に次のトレーラーを含めてください:
  ```
  Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
  ```

## ここでやってはいけないこと

- デモアプリのアプリケーションコードを追加しない — Astro の SSR エンドポイント、Drizzle データレイヤー、`.astro` UI コンポーネント、Tailwind クラス、Vitest/Playwright のテスト。これらは `github-samples/tailspin-toys` に属します。
- アプリケーションのソースパスに対して執筆しない — デモアプリはここではなく `tailspin-toys` にある単一の Astro プロジェクトです。
- タスクの最後にサマリーの Markdown ファイルを生成しない。
- `mkdocs.yml` やその他の並行するドキュメントツールを追加しない — サイトは Astro + Starlight です。
