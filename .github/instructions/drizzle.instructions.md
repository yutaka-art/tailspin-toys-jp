---
description: 'Astro アプリ向けの Drizzle ORM + Node SQLite データ層のパターン'
applyTo: 'db/**/*.ts,src/lib/*.ts'
---

# Drizzle ORM + Node SQLite に関する指示

このアプリのデータは、Node.js 組み込みの `node:sqlite` ドライバー上で **Drizzle ORM** を介してアクセスするローカルの SQLite データベースに保存されています。データは Astro ページのフロントマターから **ビルド時** に読み込まれ、実行時の API サーバーは存在しません。スキーマの変更は **drizzle-kit** のマイグレーションで管理します。

## 構成

- `db/schema.ts` — Drizzle のテーブル定義（`publishers`、`categories`、`games`）と、そこから推論される行の型。スキーマの唯一の情報源（single source of truth）です。
- `db/transforms.ts` — **純粋** 関数（CSV のパース、説明文の生成、重複排除、決定的な `ratingFromTitle`）。DB アクセスを持たないため、ユニットテストが容易です。
- `db/seed.ts` — transforms を使って `db/games.csv` から冪等（idempotent）にシードを行います。
- `db/migrate.ts` — 生成済みのマイグレーションを適用します。
- `db/migrations/` — 生成された SQL マイグレーション（手動で編集しないこと）。
- `db/test-helpers.ts` — `createTestDatabase()` はテスト用に、マイグレーション済みのインメモリ Node SQLite データベースを返します。
- `src/lib/db.ts` — `createDatabase(url)` / `getDatabase()` が `DATABASE_URL`（既定ではローカルの `tailspin.db` ファイル）から Drizzle クライアントを構築します。
- `src/lib/games.ts` — ページとテストの双方で使用する、型付きの **db 注入可能（injectable-db）** なデータアクセス用ヘルパー。

## スキーマの規約

- `sqliteTable` を使用し、カラム名を明示すること（`text`、`integer`、`real`）。
- 主キー: `integer('id').primaryKey({ autoIncrement: true })`。
- 必須カラムには `.notNull()` を付けること。null 許容カラム（例: `starRating`）は null 許容のままにします。
- 外部キーには `.references(() => other.id)` を使用すること。
- 推論された型（`typeof table.$inferSelect`）をエクスポートし、それを基にアプリ向けの型を構築すること。行の形を手作業で再定義しないこと。

## マイグレーションのワークフロー

1. `schema.ts` を編集する。
2. マイグレーションを生成する: `npm run db:generate`（drizzle-kit）。
3. ローカルで適用＋シードを行う: `npm run db:setup`（`db:migrate` + `db:seed`）。
4. スキーマの変更 **と**、`db/migrations/` に生成されたマイグレーションの両方をコミットする。

> [!IMPORTANT]
> データベースは `astro build` の **前に** マイグレーションとシードを済ませておく必要があります。`prebuild`／`predev` の npm スクリプトが `db:setup` を自動的に実行し、CI もこの順序に依存しています。

## データアクセス用ヘルパー（db 注入可能）

ヘルパーは第 1 引数として `db` インスタンスを受け取るため、実際のクライアント（ページ内）とインメモリクライアント（テスト内）の両方で動作します。

```ts
import { asc, count, eq } from 'drizzle-orm';
import type { Database } from './db';
import { games } from '../../db/schema';

export async function getAllGameIds(db: Database): Promise<number[]> {
  const rows = await db.select({ id: games.id }).from(games).orderBy(asc(games.title));
  return rows.map((r) => r.id);
}
```

- 静的ビルドを決定的にするため、常に安定したカラム（title）で `order by` すること。
- 生の行をアプリ向けの `Game`／`Publisher`／`Category` 型へのマッピングは 1 箇所にまとめること。Drizzle の行の形をコンポーネントに漏らさないこと。
- 並び替えや検索のロジックはページではなく `games.ts` に置くこと。

## 決定性（Determinism）

シードから導出される値は、ビルド間で再現可能でなければなりません。スター評価はタイトルの安定したハッシュ（`ratingFromTitle`）から導出すること。**決して** `Math.random()` を使わないこと。

## テスト

transforms は直接、ヘルパーは `createTestDatabase()` に対してユニットテストを行うこと。[`unit-tests.instructions.md`](unit-tests.instructions.md) を参照してください。

## Node.js の要件

データ層は実験的フラグなしで組み込みの `node:sqlite` モジュールを使用するため、Node.js 22.13 以降が必要です。プラットフォーム固有のバイナリを同梱するサードパーティ製の SQLite ドライバーを導入しないこと。

## 型チェック

データ層（`db/**/*.ts`、`src/lib/*.ts`）は `npm run typecheck` によって型チェックされます。これは `tsconfig.tsgo.json` を使い、ネイティブの **TypeScript 7** コンパイラ（`@typescript/native-preview` 由来の `tsgo`）を実行します。`tsgo` が検証できるよう、ヘルパーは明示的な引数と戻り値の型を付けてエクスポートしてください。Lint には影響しません。ESLint + `typescript-eslint` は引き続き従来の `typescript` パッケージ上で実行されます。
