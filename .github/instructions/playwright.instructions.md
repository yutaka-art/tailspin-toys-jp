---
description: 'Playwright テスト生成の指示'
applyTo: '**/*.spec.ts'
---

# テスト作成ガイドライン

## コード品質の基準

- **ロケーター**: 堅牢性とアクセシビリティのため、ユーザーに見える形のロール（役割）ベースのロケーター（`getByRole`、`getByLabel`、`getByText` など）を優先すること。`test.step()` を使って操作をグループ化し、テストの可読性とレポートの見やすさを高めること。
- **タイムアウト**: Playwright に組み込まれた自動待機（auto-waiting）の仕組みのみに頼ること。`waitForTimeout` のようなハードコードされた待機、デフォルトタイムアウトの延長、`waitForLoadState` は絶対に使わないこと。
- **アサーション**: 自動リトライ付きの Web ファーストなアサーションを使用すること。これらのアサーションは `await` キーワードで始まります（例: `await expect(locator).toHaveText()`）。コンテンツや構造を本当に検証したい場合は、単なる `toBeVisible()` よりも、意味のある状態を検証できるアサーション（`toHaveText`、`toContainText`、`toHaveCount`、`toMatchAriaSnapshot`、`toHaveURL`）を優先すること。`toBeVisible()` も有効な自動リトライ付きアサーションであり、純粋に存在・表示を確認する場合には適切ですが、より具体的なアサーションのほうが意図をよく表せる場面では使わないこと。
- **明確さ**: テストやステップのタイトルには、意図が明確に伝わる説明的な名前を付けること。コメントは複雑なロジックや自明でない操作を説明する場合にのみ追加すること。

## テストの構造

- **インポート**: `import { test, expect } from '@playwright/test';` から始めること。
- **構成**: ある機能に関連するテストは `test.describe()` ブロックにまとめること。
- **フック**: `describe` ブロック内のすべてのテストに共通するセットアップ処理（例: ページへの遷移）には `beforeEach` を使うこと。
- **タイトル**: `機能 - 具体的な操作やシナリオ` のように、明確な命名規則に従うこと。


## ファイルの構成

- **配置場所**: すべてのテストファイルは `e2e-tests/` ディレクトリに保存すること。
- **命名**: `<機能名またはページ名>.spec.ts` という規則を使うこと（例: `login.spec.ts`、`search.spec.ts`）。
- **範囲**: 主要なアプリ機能やページごとに 1 つのテストファイルを目安とすること。

## アサーションのベストプラクティス

- **UI 構造**: コンポーネントのアクセシビリティツリーの構造を検証するには `toMatchAriaSnapshot` を使うこと。これにより、包括的でアクセシブルなスナップショットが得られます。
- **要素数**: ロケーターで見つかった要素の数を検証するには `toHaveCount` を使うこと。
- **テキスト内容**: 完全一致には `toHaveText`、部分一致には `toContainText` を使うこと。
- **ナビゲーション**: 操作後のページ URL を検証するには `toHaveURL` を使うこと。


## テスト構造の例

```typescript
import { test, expect } from '@playwright/test';

test.describe('Movie Search Feature', () => {
  test.beforeEach(async ({ page }) => {
    // 各テストの前にアプリケーションへ遷移する
    await page.goto('https://debs-obrien.github.io/playwright-movies-app');
  });

  test('Search for a movie by title', async ({ page }) => {
    await test.step('Activate and perform search', async () => {
      await page.getByRole('search').click();
      const searchInput = page.getByRole('textbox', { name: 'Search Input' });
      await searchInput.fill('Garfield');
      await searchInput.press('Enter');
    });

    await test.step('Verify search results', async () => {
      // 検索結果のアクセシビリティツリーを検証する
      await expect(page.getByRole('main')).toMatchAriaSnapshot(`
        - main:
          - heading "Garfield" [level=1]
          - heading "search results" [level=2]
          - list "movies":
            - listitem "movie":
              - link "poster of The Garfield Movie The Garfield Movie rating":
                - /url: /playwright-movies-app/movie?id=tt5779228&page=1
                - img "poster of The Garfield Movie"
                - heading "The Garfield Movie" [level=2]
      `);
    });
  });
});
```

## 作成・反復の進め方

> [!NOTE]
> このファイルはスペック（テスト）の書き方を扱います。E2E スイートを *実行* するには `quality-checks` スキルを使用し、`npx playwright test` を直接呼び出さないこと。

1. **実行**: `quality-checks` スキルを通じてスイートを実行する。
2. **失敗のデバッグ**: テストの失敗を分析し、根本原因を特定する。
3. **反復**: 必要に応じてロケーター、アサーション、テストロジックを改善し、スキルを通じて再実行する。
4. **検証**: テストが一貫して成功し、意図した機能を確実にカバーしていることを確認する。
5. **報告**: テスト結果と発見した問題についてフィードバックを提供する。

## 品質チェックリスト

テストを確定する前に、以下を確認すること:
- [ ] すべてのロケーターがアクセシブルかつ具体的で、strict モード違反を起こさないこと
- [ ] テストが論理的にグループ化され、明確な構造に従っていること
- [ ] アサーションが意味を持ち、ユーザーの期待を反映していること
- [ ] テストが一貫した命名規則に従っていること
- [ ] コードが適切にフォーマットされ、コメントが付けられていること
