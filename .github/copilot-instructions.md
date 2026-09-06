# Tailspin Toys クラウドファンディング開発ガイドライン

これは開発者をテーマにしたゲーム向けのクラウドファンディングプラットフォームです。アプリケーションは **Astro 7** による単一サイト（完全にプリレンダリングされた静的出力）で、**Tailwind CSS v4** でスタイリングされています。データはローカルの SQLite データベースに保存され、**Drizzle ORM と Node.js 組み込みの SQLite ドライバー** を通じてビルド時にアクセスされます。各ページはフロントマターで直接データベースをクエリします。別建てのバックエンド API やクライアントサイドの UI フレームワークは存在しません。コントリビュートする際は、以下のガイドラインに従ってください。

## エージェント向けの注意事項

- コード生成を始める前に、まずプロジェクトを調査すること
- 長い作業では TODO リストを作成すること
  - TODO リストの各ステップに取りかかる前に、常に正しい指示に従えるよう、指示内容を読み直すこと
- 利用可能な場合は常に instructions ファイルを使用し、コード生成前に内容を確認すること
- タスク完了時にサマリー用の Markdown ファイルを生成しないこと
- スクリプトや BASH コマンドを実行する際は、常に絶対パスを使用すること
- **明示的に指示された場合を除き、main へのコミットやプッシュを自動で行わないこと**

## コード標準

### 各コミット前に必須の事項

#### テストのガイドライン

- **テストと Lint は必ず `quality-checks` スキルを通じて実行すること。`npm run test:unit`、`npm run test:e2e`、`npm run lint` を直接実行しないこと。** このスキルは環境セットアップ、実行順序、トラブルシューティングをまとめて扱います。（手動確認のためにアプリを起動するのは品質チェックではありません。その場合は `npm run dev` を直接実行してください。）
- Vitest のユニットテストを実行してデータ層と変換処理を検証し、Playwright のテストを実行して E2E とフロントエンドの機能を検証すること
- コミット前に ESLint を実行してフロントエンドのコード品質をチェックすること
- 既存のテストを確認し、作業が重複しないようにすること
- テストコードもプロジェクトの他の部分と同等の品質を保ち、DRY 原則に従うこと
- フロントエンドの変更については、ビルド（`npm run build`）を直接実行して検証し、`quality-checks` スキルを通じて E2E テストを実行し、すべてが正しく動作することを確認すること
- データ層（スキーマ、ヘルパー、変換処理）を変更した場合は、対応するユニットテストを更新して実行すること

#### プロジェクトのガイドライン

- データベースのスキーマを更新した場合は、drizzle-kit のマイグレーションを生成してコミットすること（`npm run db:generate`）
- 新しい機能を追加した場合は、必ず README を更新すること
- プロジェクト構成やスクリプト、プログラミングに関する指針など、関連する変更があれば Copilot Instructions ファイルのすべての記述を更新すること

### コードフォーマットの要件

- TypeScript を使用し、特にデータ層（`db/`、`src/lib/`）では関数の引数と戻り値に明示的な型を付けること
- フロントエンドのコード（TypeScript、Astro）は ESLint のチェック（`npm run lint`）を通過すること

### データ層のパターン（Drizzle + Node SQLite）

- テーブルは `db/schema.ts` で定義し、スキーマ変更は drizzle-kit のマイグレーションで管理すること（`drizzle.instructions.md` を参照）
- データアクセス用のヘルパーは `src/lib/` に置き、テスト可能にするために **注入可能な `db`** 引数を持たせること
- CSV／シード処理のロジックは `db/transforms.ts` の純粋関数として保つこと
- 静的ビルドを再現可能にするため、シードから導出される値は決定的であること（`Math.random` を使わないこと）

### Astro のパターン

- **Astro のページ／コンポーネント**: ルーティング、レイアウト、コンテンツ、コンポーネントはすべて `.astro`（`astro.instructions.md` を参照）
- データはページのフロントマターから `src/lib/` のヘルパーを介して直接クエリすること（ビルド時、静的出力）
- 動的ルートでは `getStaticPaths()` と `export const prerender = true` を使用すること
- ブランドを反映した `404.astro` を用意すること（静的出力では未知のルートは実際の 404 になります）
- 本当にクライアント側のインタラクティブ性が必要な場合にのみ、スコープ付きの Astro `<script>` を追加すること

### スタイリング

- Tailwind CSS のユーティリティクラスのみを使用すること（`style.instructions.md` を参照）
- ダークテーマの配色: slate パレット（`bg-slate-800`、`text-slate-100` など）
- 角丸とモダンな UI パターンを使用すること
- クリーンでアクセシブルなインターフェースを備えた、モダンな UI/UX の原則に従うこと

### GitHub Actions ワークフロー

- 適切なセキュリティプラクティスに従うこと
- ワークフローの権限（permissions）を明示的に設定すること
- どのようなタスクを実行しているかを説明するコメントを追加すること

## スクリプト

- このプロジェクトはすべての開発タスクに **npm スクリプト** を使用します。`scripts/` ディレクトリは存在しません。
- **スキルが優先されます。** コマンドを直接実行する前に、そのタスクをカバーするスキルがないか確認してください（例: `quality-checks` スキルはテストと Lint をまとめて扱います）。該当するものがあれば、それに従ってください。
- 主な npm スクリプト:
  - `npm run dev` — Astro の開発サーバーを起動（`predev` がローカル SQLite データベースのマイグレーションとシードを実行）
  - `npm run build` — 静的サイトをビルド（`prebuild` がローカル SQLite データベースのマイグレーションとシードを実行）
  - `npm run preview` — ビルド済みの `dist/` 出力を配信
  - `npm run lint` — ESLint
  - `npm run test:unit` — Vitest のユニットテスト
  - `npm run test:e2e` — Playwright の E2E テスト（先にビルドとプレビューを実行）
  - `npm run typecheck` — `tsgo`（`@typescript/native-preview` 経由の TypeScript 7 ネイティブコンパイラ）で `tsconfig.tsgo.json` を使い、純粋な TypeScript を型チェック
  - `npm run typecheck:astro` — `astro check`（従来の TypeScript パッケージ）で `.astro` ファイルを型チェック
  - `npm run typecheck:all` — 両方の型チェックスクリプトを実行（CI の `type-check` ジョブで使用）
  - `npm run db:generate` / `db:migrate` / `db:seed` / `db:setup` — Drizzle のスキーマ／マイグレーション／シードタスク

> [!NOTE]
> TypeScript 7（`tsgo`）は型チェック専用として **併用** で採用しており、Lint には影響しません。ネイティブコンパイラの API がまだ対応していないため、ESLint + `typescript-eslint` と `astro check` は引き続き従来の `typescript` パッケージ（v6 に固定）を解決に使用します。`typescript-eslint` と `@astrojs/check` がネイティブ API に対応するまでは、従来の `typescript` パッケージを 7 に上げないでください（Dependabot の `ignore` で固定されています）。`tsgo` は `--noEmit` 専用であり、サイトのビルドは引き続き `astro build` が担当します。

## リポジトリ構成

アプリケーションはリポジトリのルートに配置されています。

- `db/`: Drizzle のスキーマ、マイグレーション、変換処理、シード、`games.csv`
- `src/lib/`: Node SQLite クライアント（`db.ts`）とデータアクセス用ヘルパー（`games.ts`）
- `src/components/`: 再利用可能な `.astro` コンポーネント
- `src/layouts/`: Astro のレイアウトテンプレート
- `src/pages/`: Astro のページルート（一覧の `index.astro`、`game/[id].astro`、`404.astro`、`about.astro`）
- `src/styles/`: CSS と Tailwind の設定
- `src/types/`: TypeScript のインターフェース（Game、Publisher、Category）
- `e2e-tests/`: Playwright の E2E テスト（ホーム、ゲーム、アクセシビリティ）
- `drizzle.config.ts`、`vitest.config.ts`、`astro.config.mjs`、`playwright.config.ts`: ツール設定
- `README.md`: プロジェクトのドキュメント
