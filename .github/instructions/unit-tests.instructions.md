---
description: 'Astro + Drizzle/Node SQLite データ層向けの Vitest ユニットテストガイドライン'
applyTo: '**/*.test.ts'
---

# ユニットテストガイドライン（Vitest + Drizzle/Node SQLite）

ユニットテストは **Vitest**（`npm run test:unit`）で実行します。フレームワークに依存しない、価値の高い次の 2 つの層を対象とします。

1. **純粋な変換処理**（`db/transforms.ts`）— CSV のパース、説明文の組み立て、重複排除、決定的な評価値の算出。
2. **データアクセス用ヘルパー**（`src/lib/games.ts`）— 並び替えや検索処理を、実際のインメモリの **Node SQLite** データベースに対して検証します。

> [!IMPORTANT]
> テストは Astro のランタイムから独立させてください。ヘルパーは **注入可能な `db`** 引数を受け取ります。テストではインメモリのデータベースを渡し、ページでは実際のクライアントを渡します。データロジックのユニットテストのために Astro サーバーを起動しないでください。

## ファイル構成

- テストは対象のコードと同じ場所に置くこと: `transforms.test.ts` は `transforms.ts` の隣に、`games.test.ts` は `games.ts` の隣に配置します。
- 命名規則: `<モジュール名>.test.ts`。
- `describe('<モジュール / 関数>')` ブロックと `it('Y のとき X をする')` 形式のケースを使用すること。
- ヘルパーやフィクスチャには型注釈を付けること。このコードベースでは明示的な型付けが必須です。

## 純粋な変換処理のテスト

- データベースは不要です。関数をインポートして、その出力を検証します。
- カバーすべき観点: 正常系、空文字・空白のみの入力、任意フィールドが欠けている行、重複排除、そして **決定性**（例: `ratingFromTitle` は同じタイトルに対して常に同じ値を返し、3.0〜5.0 の範囲に収まる）。
- 入力／出力の組み合わせには `it.each` を使ったテーブル駆動のケースを優先すること。

```ts
import { describe, it, expect } from 'vitest';
import { ratingFromTitle } from './transforms';

describe('ratingFromTitle', () => {
  it('is deterministic and within range', () => {
    const a = ratingFromTitle('Code Quest');
    const b = ratingFromTitle('Code Quest');
    expect(a).toBe(b);
    expect(a).toBeGreaterThanOrEqual(3.0);
    expect(a).toBeLessThanOrEqual(5.0);
  });
});
```

## データアクセス用ヘルパーのテスト

- テストごとに、共有ヘルパー `createTestDatabase()`（`db/test-helpers.ts`）を使って新しいインメモリデータベースを構築すること。このヘルパーは `:memory:` の Node SQLite クライアント上でマイグレーションを実行します。
- そのテストに必要なフィクスチャだけをシードし、その `db` を渡してヘルパーを呼び出すこと。
- 深いオブジェクトの構造を検証する前に、まず軽い項目（件数、合計、並び順）を先に検証すること。

```ts
import { describe, it, expect, beforeEach } from 'vitest';
import { createTestDatabase } from '../../db/test-helpers';
import { getAllGames, getGameById } from './games';

describe('getAllGames', () => {
  let db: Awaited<ReturnType<typeof createTestDatabase>>;

  beforeEach(async () => {
    db = await createTestDatabase();
    // …publishers、categories、games をシードする…
  });

  it('returns games ordered by title with their relations', async () => {
    const games = await getAllGames(db);
    const titles = games.map((g) => g.title);
    expect(titles).toEqual([...titles].sort());
    expect(games[0].category).not.toBeNull();
  });
});
```

## 必須のカバレッジ

- 有効なデータでの成功ケース
- 見つからないケース（存在しない id を渡した `getGameById` は `null` を返す）
- データベース／コレクションが空のシナリオ
- 並び順の保証（タイトルのアルファベット順）— 静的ビルドはこれが決定的であることに依存しています
- シードから導出される値の決定性

## ベストプラクティス

- Arrange-Act-Assert（準備・実行・検証）の流れに従うこと。
- 1 つの `it` につき 1 つの振る舞いを検証すること。1 つのケースで無関係な項目を検証しないこと。
- データベースをモックしないこと。インメモリの Node SQLite インスタンスは高速で、実際の SQL／JOIN を検証できます。
- フィクスチャは最小限にしつつ、リレーション（game → publisher、game → category）を代表できるものにすること。
- スキーマ変更でテストが壊れた場合は、`npm run db:generate` でマイグレーションを再生成し、フィクスチャを更新すること。
