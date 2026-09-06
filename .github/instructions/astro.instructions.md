---
description: 'ページ、レイアウト、コンポーネント、ルーティング向けの Astro コンポーネントのパターン'
applyTo: '**/*.astro'
---

# Astro コンポーネントに関する指示

## Astro コンポーネントのパターン

Astro は UI に関するすべて（ページ、レイアウト、コンポーネント、ルーティング、コンテンツ）を担います。このサイトは **完全にプリレンダリング** されており（`output: 'static'`）、クライアントサイドの UI フレームワークも別建ての API サーバーも存在しません。各ページはビルド時に、`src/lib/` にある Drizzle／Node SQLite のデータアクセス用ヘルパーを介して **フロントマターで直接** データを読み込みます。

### コンポーネントの構造

```astro
---
// フロントマター: ビルド時に実行される（静的出力）
import Layout from '../layouts/Layout.astro';
import GameCard from '../components/GameCard.astro';
import { getDatabase } from '../lib/db';
import { getAllGames } from '../lib/games';

interface Props {
  title: string;
}

const { title } = Astro.props;
const games = await getAllGames(getDatabase());
---

<Layout title={title}>
  {games.map((game) => <GameCard {game} />)}
</Layout>
```

## レイアウト

- 再利用可能なレイアウトコンポーネントは `src/layouts/` に作成すること
- コンテンツの差し込みには `<slot />` を使用すること
- 共通要素（`<head>`、ナビゲーション、フッター）を含めること
- グローバルスタイルはレイアウトでインポートすること

### レイアウトの例

```astro
---
interface Props {
  title: string;
}
const { title } = Astro.props;
---

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>{title}</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

## ページ

- ページは `src/pages/` に作成すること
- ファイルベースのルーティング: `src/pages/about.astro` → `/about`
- 動的ルート: `src/pages/game/[id].astro`
- ブランドを反映した `src/pages/404.astro` を用意すること。静的出力では、生成されたページが存在しない URL は実際の 404 になります。

### 動的ルート（静的出力）

`output: 'static'` では、すべての動的ルートは `getStaticPaths()` でページを列挙し、`prerender = true` を設定する必要があります。データはデータアクセス用ヘルパーを使ってフロントマターでクエリします。

```astro
---
import type { GetStaticPaths } from 'astro';
import Layout from '../../layouts/Layout.astro';
import { getDatabase } from '../../lib/db';
import { getAllGameIds, getGameById } from '../../lib/games';

export const prerender = true;

export const getStaticPaths = (async () => {
  const ids = await getAllGameIds(getDatabase());
  return ids.map((id) => ({ params: { id: String(id) } }));
}) satisfies GetStaticPaths;

const { id } = Astro.params;
const game = await getGameById(getDatabase(), Number(id));
---

<Layout title="Game Details - Tailspin Toys">
  <!-- ゲームの詳細 -->
</Layout>
```

## データアクセス

- ビルド時のデータは、**Drizzle ORM + Node SQLite** を介してローカルの SQLite データベースから取得します（[`drizzle.instructions.md`](drizzle.instructions.md) を参照）。
- `getDatabase()` は `src/lib/db.ts` から、型付きヘルパーは `src/lib/games.ts` からインポートすること。
- データベースは `astro build` の前にマイグレーションとシードを済ませておく必要があります。`prebuild` の npm スクリプト（`db:setup`）がこれを行います。

## クライアントのインタラクティブ性（まれ）

Svelte／React のレイヤーは存在しません。ページに本当にクライアント側の挙動が必要な場合は、標準の DOM API を使ったスコープ付きの Astro `<script>` を追加すること。キーボードやフォーカスの挙動を無償で得られるよう、ネイティブのインタラクティブ要素（`<button>`、`<a href>`）を優先すること。

## TypeScript

- 型安全な props のために TypeScript を使用すること
- `Props` インターフェースをフロントマターで定義すること
- コンポーネントのインポートとヘルパーの戻り値に型を付けること
- Lint や型チェックの前に `npx astro sync` を実行して、ルート／コンテンツの型を（再）生成すること
- `.astro` ファイルは `npm run typecheck:astro`（`astro sync` の後に `astro check` を実行）によって、従来の `typescript` パッケージ上で型チェックされます。`db/`、`src/lib/`、`src/types/` にある純粋な TypeScript は、`npm run typecheck`（ネイティブの TS 7 コンパイラ `tsgo`）によって別途型チェックされ、こちらは `.astro` ファイルを **処理しません**。

## ベストプラクティス

- データの取得はフロントマター（ビルド時）にとどめ、クライアントサイドでの取得は避けること
- クライアントサイドの JavaScript を最小限にすること。既定では JS は一切出力されません
- グローバルな CSS スタイルはレイアウトからインポートして使用すること
- インタラクティブ要素には常に `data-testid` を付けること（`ui.instructions.md` を参照）
