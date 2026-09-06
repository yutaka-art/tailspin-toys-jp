---
name: quality-checks
description: このプロジェクトのすべてのテスト・Lint・品質チェックの実行を担当します — Vitest のユニットテスト、Playwright の E2E テスト、ESLint の実行、失敗のデバッグ、コード変更の検証、そしてコミット・プッシュ・マージ前の準備完了確認を行います。テスト・Lint・検証コマンド（`npm run test:unit`、`npm run test:e2e`、`npm run lint` など）を直接実行する代わりに、このスキルを使用してください。
allowed-tools:
  - shell
---

# 品質チェック

これは単一の Astro アプリケーション（Astro 7 + Drizzle ORM／Node SQLite）です。すべてのコマンドは npm スクリプトを通じてリポジトリのルートから実行します。

## クイックリファレンス

| テストスイート | コマンド | 使用するタイミング |
|------------|----------------------------|-------------|
| ユニットテスト（Vitest） | `npm run test:unit` | データ層／変換処理／ヘルパーを変更した後 |
| フロントエンド E2E テスト（Playwright） | `npm run test:e2e` | UI／ページ／コンポーネントを変更した後 |
| Lint（ESLint） | `npm run lint` | TypeScript または Astro を変更した後 |
| 型チェック（tsgo + astro check） | `npm run typecheck:all` | TypeScript または Astro を変更した後 |

すべてのコマンドは、依存関係がインストールされている（`npm ci`）ことを前提とします。E2E の場合は、Playwright の Chromium ブラウザが利用可能である（`npx playwright install chromium`）ことも前提です。

---

## 検証スイートの実行

### ユニットテスト

```bash
npm run test:unit
```

- `db/**/*.test.ts` と `src/**/*.test.ts` に対して Vitest（`vitest run`）を実行します。
- 純粋なシード／変換関数と、Drizzle のデータアクセス用ヘルパーを、インメモリの Node SQLite データベースを使って検証します。

### フロントエンド E2E テスト

```bash
npm run test:e2e
```

- Playwright の `webServer` は、まず静的サイトを**ビルド**し（`prebuild` スクリプトが `db:migrate` + `db:seed` を実行）、`astro preview` でポート 4321 上に配信します。
- ビルド済みの `dist/` 出力に対して、`e2e-tests/` 内のすべての Playwright スペック（ホームページ、ゲーム一覧／詳細ページ、アクセシビリティ、404）を実行します。

### Lint

```bash
npm run lint
```

- プロジェクト内のすべての TypeScript および Astro ファイルに対して ESLint を実行します。
- コミット前にエラーがゼロで通過する必要があります。

### 型チェック

```bash
npm run typecheck:all
```

- `npm run typecheck` は、ネイティブの **TypeScript 7** コンパイラ（`@typescript/native-preview` の `tsgo`）を使い、`tsconfig.tsgo.json`（`--noEmit`）を通じて純粋な TypeScript（`db/`、`src/lib/`、`src/types/`、各種設定、テスト）を型チェックします。
- `npm run typecheck:astro` は `astro sync` を実行した後、`.astro` ファイルに対して `astro check` を実行します（従来の `typescript` パッケージを使用）。
- 型チェックは Lint とは独立しています。`tsgo` は ESLint に影響せず、ESLint は引き続き従来の `typescript` パッケージを使用します。コミット前にはどちらもエラーゼロで通過する必要があります。

---

## デバッグとトラブルシューティング

### 環境／セットアップの失敗

**症状**: `command not found`、モジュールの欠落、または `Cannot find package`。

```bash
npm ci
npx playwright install --with-deps chromium   # E2E の場合のみ必要
```

- Node 22.13 以降が利用可能か確認してください: `node --version`。
- エディターや型のエラーが、生成された Astro の型が見つからないことを示す場合は、`npx astro sync` を実行してください。

---

### データベース／ビルド時のデータ

**症状**: ページが空になる、`no such table`、またはビルドでゲームページが生成されない。

SQLite データベースは、`astro build` の**前に**マイグレーションとシードを行う必要があります。`prebuild`／`predev` スクリプトがこれを自動で実行しますが、手動で実行することもできます:

```bash
npm run db:setup     # db:migrate + db:seed
```

- データベースは `tailspin.db`（gitignore 対象）に配置され、`db/games.csv` から再生成されます。
- クリーンな再ビルドを強制するには: `rm -f tailspin.db && rm -rf dist && npm run build`。

---

### ポートの競合

**症状**: ポート 4321 で `Address already in use`。

```bash
lsof -ti :4321 | xargs kill
```

その後、失敗したコマンドを再実行してください。別のチェックアウトから残った古い `astro dev`／`astro preview` サーバーに注意してください。Playwright はローカルで 4321 上の既存サーバーを再利用します。

---

### Playwright／E2E テストの失敗

**症状**: テストのタイムアウト、要素が見つからない、または HTTP ステータスが不正。

1. **ブラウザ未インストール**: `npx playwright install --with-deps chromium`。
2. **古いサーバーの再利用**: 4321 上に残った dev／preview サーバーが古い HTML を配信することがあります。それを停止し（ポートの競合を参照）、`webServer` が再ビルドするよう再実行してください。
3. **ロケーターの変更**: `data-testid` がリネームまたは削除された場合は、スペックを合わせて更新してください。
4. **404 の期待値**: 存在しないゲーム ID（例: `/game/99999`）は、静的出力では**実際の 404** になります。ページ内のエラーメッセージではなく、`not-found` の testid を検証してください。
5. **不安定なテスト**: ハードコードされた待機を、自動リトライされる web-first アサーションに置き換えてください（[playwright.instructions.md](../../instructions/playwright.instructions.md) を参照）。**`waitForTimeout` は決して使用しないでください。**

反復を速くするために単一のスペックを実行するには:

```bash
npx playwright test e2e-tests/games.spec.ts
```

---

### ユニットテストの失敗

**症状**: `npm run test:unit` でのアサーション失敗。

1. **失敗したアサーションを読む** — Vitest は期待値と実際の値をインラインで出力します。
2. **インメモリデータベース**: ヘルパーのテストは、テストごとに新しい `:memory:` の Node SQLite データベースを構築し、マイグレーションを実行してフィクスチャをシードします。スキーマの変更が反映されない場合は、`npm run db:generate` でマイグレーションを再生成してください。
3. **決定性**: 星評価はタイトルの安定したハッシュ（`ratingFromTitle`）から導出されます。`Math.random` は決して使いません。評価のアサーションが不安定な場合は、通常、非決定的なデータが紛れ込んでいることを意味します。

単一のファイルを実行するには:

```bash
npx vitest run src/lib/games.test.ts
```

---

### Lint の失敗

**症状**: `npm run lint` からの ESLint エラー。

1. **安全な問題を自動修正**: `npm run lint -- --fix`。
2. **未使用の変数**: 意図的に未使用とする識別子には `_` を接頭辞として付けてください。
3. **TypeScript の型エラー**: 欠けている型注釈を追加するか、誤った型を修正してください。
4. **`--fix` 後に残るエラー**: 手動で解決してください。正当な理由なく `eslint-disable` で抑制しないでください。

---

### ローカルと CI の差異

**症状**: ローカルではテストが通るが CI では失敗する（またはその逆）。

- **Node バージョンの不一致**: CI は現行の Node LTS リリースを使用します。
- **データベースの状態**: CI は常にクリーンなシードからビルドします。ローカルで古いデータが疑われる場合は、`tailspin.db` を削除して再ビルドしてください。
- **ビルド版と dev 版の違い**: CI は `astro preview` を介してビルド済みの `dist/` をテストします。`astro dev` に対してではなく、（先にビルドを行う）`npm run test:e2e` を使ってローカルで再現してください。

---

## 検証ポリシー

### コミット／マージ前にテストが通過している必要があります

- 変更をコミットする前に、既存のすべてのテストが通過している必要があります
- 正当な理由なくテストをスキップまたは無効化しないでください
- 壊れたテストはマージをブロックします。無視せず修正してください
- 変更したコードのテストだけでなく、テストスイート全体を実行してください
- 新機能には適切なテストカバレッジを付けて提供する必要があります

> [!NOTE]
> このスキルはテストの**実行・検証・デバッグ**を対象とします。テストコードの**書き方**（構造、フィクスチャ、命名、ロケーター、品質基準）については、唯一の信頼できる情報源である instructions ファイルに従ってください:
> - ユニットテスト（`**/*.test.ts`）: [unit-tests.instructions.md](../../instructions/unit-tests.instructions.md)
> - フロントエンド E2E（`e2e-tests/*.spec.ts`）: [playwright.instructions.md](../../instructions/playwright.instructions.md)

---

## コミット前チェックリスト

1. Lint を実行する（フロントエンドのファイルを変更した場合）: `npm run lint`
2. 型チェックを実行する（TypeScript／Astro ファイルを変更した場合）: `npm run typecheck:all`
3. ユニットテストを実行する（データ層／ヘルパーを変更した場合）: `npm run test:unit`
4. E2E テストを実行する（UI を変更した場合）: `npm run test:e2e`
5. 新機能に適切なテストカバレッジがあることを確認する
6. テストが壊されたり、スキップまたは無効化されていないことを確認する
