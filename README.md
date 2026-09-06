# Tailspin Toys

Tailspin Toys は、開発者向けをテーマにしたゲームのクラウドファンディングプラットフォームです。本プロジェクトは架空のゲームクラウドファンディング企業のWebサイトであり、[Astro](https://astro.build/) を用いた単一サイト（完全にプリレンダリングされた静的出力）として構築され、[Tailwind CSS](https://tailwindcss.com/) でスタイリングされています。データはローカルの SQLite データベースに保存され、[Drizzle ORM](https://orm.drizzle.team/) と Node.js 組み込みの SQLite ドライバーを介してアクセスします。各ページはビルド時にフロントマターから直接データベースへクエリを実行するため、独立したバックエンドサービスは存在しません。

## アーキテクチャ

- **Astro 7** — ページ、レイアウト、コンポーネント、ルーティングを担当します。`output: 'static'` を使用しているため、サイト全体がビルド時に HTML へプリレンダリングされます。
- **Drizzle ORM + Node SQLite** — データ層です。スキーマは `db/schema.ts` に定義され、データは `db/games.csv` からシードされます。マイグレーションは `drizzle-kit` で管理します。
- **Tailwind CSS v4** — ユーティリティクラスによるスタイリング（ダークテーマ）です。
- **Vitest** — データ層と純粋な変換処理のユニットテストです。
- **Playwright** — ビルド済みの静的サイトに対して実行するエンドツーエンドテストです。

データベースは `dev`/`build` の前に（`predev`/`prebuild` の npm スクリプトを通じて）自動的にマイグレーションおよびシードされ、gitignore 対象の `tailspin.db` ファイルに書き込まれます。

## このテンプレートの利用

このリポジトリは GitHub テンプレートです。テンプレートから新しいリポジトリを作成すると、一度だけ実行される **Bootstrap template issues** ワークフロー（`.github/workflows/bootstrap-issues.yml`）が `main` への最初のプッシュ時に自動的に実行され、最初に取り組むおすすめ機能を説明する一連のスターター issue を作成します。各 issue は `.github/bootstrap-issues/` 内の Markdown ファイルで定義されており、最初の見出しが issue のタイトルに、残りの内容が本文になります。これらのファイルを編集・追加・削除することで、作成される issue を制御できます。

このワークフローはテンプレートから作成されたリポジトリでのみ実行されます（`if: ${{ !github.event.repository.is_template }}` のガードによってテンプレート自身では実行されません）。issue の作成後は、クリーンアップコミットでワークフロー自身と `.github/bootstrap-issues/` フォルダーを削除するため、二度と実行されることはありません。

## はじめに

Node.js 22.13 以降を使用して、依存関係を一度だけインストールします。

```bash
npm ci
npx playwright install chromium   # E2E テストの実行時にのみ必要です
```

## サイトの起動

```bash
npm run dev
```

`predev` が最初にローカルデータベースのマイグレーションとシードを行います。その後、[Webサイト](http://localhost:4321) にアクセスするとサイトを確認できます。

代わりに本番ビルドをプレビューする場合は、次のようにします。

```bash
npm run build      # prebuild がマイグレーション + シードを行い、静的サイトをビルドします
npm run preview
```

## データベース

SQLite データベースは `db/games.csv` から構築されます。移行が必要なライブデータはありません。

```bash
npm run db:generate   # db/schema.ts の編集後にマイグレーションを生成します
npm run db:migrate    # マイグレーションを適用します
npm run db:seed       # games.csv からシードします（冪等）
npm run db:setup      # マイグレーション + シード（predev/prebuild により自動実行されます）
```

> [!NOTE]
> シード処理は冪等です。変更された行を照合し直すのではなく、（タイトルで一致する）既に存在するゲームをスキップします。CI は常にクリーンなデータベースから開始するため、`games.csv` を正確に反映します。ローカルで `games.csv` の行を編集または削除した場合は、`tailspin.db` を削除して `npm run db:setup` を再実行し、完全に再生成してください。

## テストの実行

```bash
npm run test:unit   # Vitest ユニットテスト（変換処理 + データアクセスヘルパー）
npm run test:e2e    # Playwright E2E テスト（先に静的サイトをビルド + プレビューします）
```

## Lint

フロントエンドでは ESLint を使用して、TypeScript と Astro ファイル全体のコード品質を担保します。次のコマンドで実行します。

```bash
npm run lint
```

ESLint は `main` へのプルリクエスト時に CI でも自動的に実行されます。

## 型チェック

本プロジェクトは型チェックに **TypeScript 7**（ネイティブの Go コンパイラー `tsgo`）を使用しており、[`@typescript/native-preview`](https://www.npmjs.com/package/@typescript/native-preview) パッケージを通じて併用しています。従来の `typescript` パッケージは意図的に v6 に固定しています。これは ESLint + `typescript-eslint` と `astro check` を変更なしで動作させ続けるためで、TypeScript 7 のプログラム的 API はまだこれらのツールに対応していないためです。

```bash
npm run typecheck        # tsgo（TS 7）が純粋な TypeScript を型チェックします（db/、src/lib/、src/types/、各種設定、テスト）
npm run typecheck:astro  # astro sync + astro check が .astro ファイルを型チェックします（従来の TypeScript パッケージを使用）
npm run typecheck:all    # 上記の両方を実行します
```

`tsgo` は [`tsconfig.tsgo.json`](tsconfig.tsgo.json) に対して実行されます。これは `.astro` ファイル（ネイティブコンパイラーが解釈できません）を除外したスコープ付きの設定です。型チェックは `main` へのプルリクエスト時に CI で自動的に実行されます。

> [!NOTE]
> ネイティブコンパイラーは型チェック（`--noEmit`）にのみ使用され、サイトのビルドは引き続き `astro build`（Vite/esbuild）が行います。従来の `typescript` パッケージは、`typescript-eslint` と `@astrojs/check` がネイティブ API（TS 7.1 頃）に対応するまで v6 のままにします。それまでの間、`.github/dependabot.yml` の Dependabot `ignore` 設定が従来の `typescript@7` へのアップグレードを保留します。

## Copilot エージェントとスキル

本プロジェクトには、品質保証を支援する Copilot のカスタマイズが含まれています。

### Database Explorer キャンバス

共有の **Database Explorer** キャンバス（`.github/extensions/database-explorer/`）は、プロジェクトの SQLite テーブルを閲覧し、一度に1つの読み取り専用 `SELECT` または `WITH` クエリを実行するための小さな UI とエージェントアクションを提供します。`.data/tailspin.db`（または `DATABASE_URL` が設定されている場合はそちら）のデータベースを使用するため、新規チェックアウトで開く前に `npm run db:setup` を実行してください。

### PR Readiness エージェント

**PR Readiness** エージェント（`.github/agents/pr-readiness.md`）は、PR 作成前の品質ゲートです。プルリクエストを作成する前に呼び出すことで、次のことを行います。

- すべての受け入れ基準が実装されているか検証します
- テストカバレッジを監査し、不足している箇所を補完します
- 完全な検証スイート（ユニットテスト、Lint、E2E テスト）を実行します
- Playwright MCP を介してブラウザーで機能を手動検証します（毎回必須）
- 実施可否（go/no-go）のレポートを作成します

### quality-checks スキル

**quality-checks** スキル（`.github/skills/quality-checks/SKILL.md`）は、プロジェクトの npm のテスト・Lint コマンドを、詳細なデバッグとトラブルシューティングの手順書とともにラップします。次のような場面で `/quality-checks` から使用します。

- セットアップ後に初めてテストや Lint を実行するとき
- テストの失敗を診断するとき（ポートの競合、残存したサーバー、不安定なテスト、CI との差異など）
- コミット、プッシュ、マージの前に準備状況を検証するとき

### GitHub Copilot アプリの実行メニュー

[GitHub Copilot アプリ](https://github.com/github/github-app) は
`.github/github-app.yml` を読み取り、**Run** メニューにプロジェクトコマンドを提供します。
新しいセッションでは依存関係が自動的にインストールされます。**Run development site** を使用すると
Astro が起動します。Astro がローカル URL を報告すると、アプリはそれを自動的にブラウザーキャンバスで開きます。
このメニューでは、オンデマンドでの検証用に静的ビルドと型チェックのコマンドも提供されます。

## ライセンス

本プロジェクトは MIT オープンソースライセンスの条件の下でライセンスされています。完全な条項については [LICENSE](./LICENSE) を参照してください。

## メンテナー

メンテナーの一覧は [CODEOWNERS](./.github/CODEOWNERS) に記載されています。

## サポート

本プロジェクトは現状のまま提供され、時間の経過とともに更新される場合があります。ご質問がある場合は issue を作成してください。

## 免責事項

このアプリは本番環境での使用を想定しておらず、本番アプリのあるべき姿を示す例として構築されたものでもありません。
