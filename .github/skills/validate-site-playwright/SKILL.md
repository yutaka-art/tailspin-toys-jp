---
name: validate-site-playwright
description: Render-validate the built Copilot Workshops site in a real browser using the Playwright MCP server. Use as an optional deeper QA pass — after the static checks in build-and-verify-docs — to confirm pages actually render: navigate the built routes, assert HTTP 200, catch console/hydration errors, find broken images, confirm Starlight Markdown features rendered, and optionally screenshot key pages. Trigger when asked to "validate the built site", "check the pages render", "do a browser/visual QA pass", or before opening a PR that changes rendered output.
---

# Playwright でビルド済みサイトを検証する

`build-and-verify-docs` はサイトを *静的に* チェックします — ビルドし、ページ数の不変条件を確認し、lychee で HTML のリンクをチェックします。ブラウザでページを開くことはないため、実行時の失敗（コンソール / ハイドレーションエラー、読み込み時に 404 になる画像、表示がおかしいレンダリング済み Markdown）は検出できません。

このスキルは **オプションの、より深い、ブラウザベースの検証** です。**Playwright MCP サーバー** を、ビルドされたサイトのローカルプレビューに対して実行します。**インタラクティブ / ローカル専用** です — CI（`pages.yml`）にはブラウザのステップがないため、これがマージのゲートになることはありません。ページのレンダリング方法を変更する PR（新しいサイトシェルコンポーネント、画像の多いレッスン、レイアウト変更）の前、または実際のレンダリング出力を確認したいときに実行してください。

ステップで特に指定がない限り、すべてのコマンドは **リポジトリルート** から実行してください。

## 前提条件

- Playwright MCP サーバーが利用可能であること（エージェントが `browser_*` ツールを持つこと）。
- クリーンビルドが存在すること。直前にビルドしていない場合は、まず [`build-and-verify-docs`](../build-and-verify-docs/SKILL.md) のビルドを実行します:

  ```bash
  cd website && rm -rf dist && npm run build && cd ..
  ```

## 1. ビルド済みサイトを配信する

実際に出荷されるものをそのまま検証できるよう、（開発サーバーではなく）本番ビルドを配信します。`npm run preview` は `website/dist/` を実際のベースパスで配信します:

```bash
cd website && npm run preview
```

サイトは <http://localhost:4321/copilot-workshops/> で配信されます。ナビゲート中も実行を継続できるよう **デタッチされたバックグラウンドプロセス** として起動し、後片付け用に PID を控えておきます。ナビゲートする前に、サーバーがリッスン中であるとログに出るまで待ちます。

## 2. チェックするルートを導き出す

URL をハードコードしないでください。どのルートが存在するかについては、ビルド済みの `dist/` が信頼できる情報源（source of truth）です:

```bash
# ベース配下のサイト絶対パスとして、ビルドされたすべてのルート
find website/dist -name index.html | grep -v 404 | sed 's#website/dist#/copilot-workshops#; s#/index.html#/#'
```

すべてのレイアウトとハーネスをカバーする **代表的なサンプル** を検証します: ランディングページ（`/copilot-workshops/`）、ハーネスごとの前提条件ページ（例: `cli/0-prerequisites/`）、そして `cli/`、`vscode/`、`cloud/`、`app/` の各々から少なくとも 1 つのレッスン。リリース検証や、共有レイアウト / コンポーネントに影響する変更の場合は、**すべての** ルートを検証します。

## 3. 各ルートを検証する

選んだ各ルートについて、Playwright MCP ツールを使います:

1. **ナビゲート** — `browser_navigate` で `http://localhost:4321/copilot-workshops/<route>` へ。
2. **レンダリングを確認** — `browser_snapshot` で、ページに見出し / タイトルと実際のコンテンツ（エラーページやレンダリングされていない生の Markdown ではないもの）があることを確認します。
3. **コンソールをチェック** — `browser_console_messages` を `level: "error"` で実行します。エラーは **ゼロ** であるべきです。ハイドレーション警告やアセットの 404 はここに現れます。

   *既知の良性な例外:* 従来のリダイレクトルート `/copilot-workshops/shared/0-prereqs/` は最小限の完全な HTML リダイレクトページで（meta refresh で即座にホームページ `/copilot-workshops/` に転送します）、favicon を宣言していないため、ブラウザが `/favicon.ico` を自動リクエストし、1 つの `404 (Not Found)` をログに出します。**リダイレクトページに限って** は、その favicon の 404 は予期されるものです。このルートは、コンソールのクリーンさではなく、ホームページに到達することを確認して検証します。*実際の* ページで favicon の 404 が出た場合は本物の問題です（実際のページは `favicon.svg` をリンクしています）。
4. **画像切れを見つける** — `browser_evaluate` で、**まず遅延読み込み画像を強制的に読み込んでから**、本当に失敗したものだけをフラグする非同期関数を使います。Starlight/Astro はフォールド下の画像に `loading="lazy"` を付けるため、単純な `naturalWidth === 0` のチェックでは、まだスクロールされて表示領域に入っていないだけの画像を偽陽性として報告してしまいます:

   ```js
   async () => {
     const imgs = Array.from(document.images);
     const results = await Promise.all(imgs.map(img => new Promise(resolve => {
       img.loading = 'eager'; // defeat lazy-loading so the asset actually fetches
       if (img.complete) return resolve(img.naturalWidth === 0 ? (img.currentSrc || img.src) : null);
       const done = ok => resolve(ok ? null : (img.currentSrc || img.src));
       img.addEventListener('load', () => done(true), { once: true });
       img.addEventListener('error', () => done(false), { once: true });
       setTimeout(() => done(img.naturalWidth > 0), 5000); // timeout guard
     })));
     return results.filter(Boolean);
   }
   ```

   空の配列は合格です。エントリがあれば、それは本当に壊れている / 欠落している画像です — 多くの場合、名前変更で失われた `_images/` パスです（[`build-and-verify-docs`](../build-and-verify-docs/SKILL.md) の整合性チェックと照合してください）。ヒットした場合は、本当の失敗として扱う前に、アセット URL に対して `curl -o /dev/null -w '%{http_code}'` で確認してください — `200` なら、壊れた画像ではなく遅延読み込みのタイミングによるものです。
5. **Markdown がきれいにレンダリングされたことを確認** — GitHub の admonition は、リテラルなテキストではなく、スタイル適用された Starlight のコールアウトとしてレンダリングされるべきです。スナップショットで、本文に見える `[!NOTE]`、`[!TIP]`、`[!CAUTION]`、`[!WARNING]`、`[!IMPORTANT]`、残った `:::` ディレクティブ、生のフロントマターがないことを確認します。
6. **スクリーンショット（オプション）** — `browser_take_screenshot` で主要ページの視覚的な記録を残します。スクリーンショットはコンテンツとしてではなく、使い捨てのアーティファクトとしてリポジトリ外に保存します。

## 4. 後片付け

終了したら、ブラウザを閉じてプレビューサーバーを停止します:

- `browser_close` でブラウザを解放します。
- ステップ 1 で起動したプレビューサーバーは `kill <PID>` で停止します。

## 報告すべき内容

ルートごとに要約します: レンダリングされたか（はい / いいえ）、コンソールエラー（件数）、画像切れ（リスト）、Markdown がきれいにレンダリングされたか（はい / いいえ）。クリーンな合格とは、**すべてのルートがレンダリングされ、コンソールエラーゼロ、画像切れゼロ、すべての Markdown がきれいにレンダリングされている** ことです。それ以外は、PR の前に修正できるよう、具体的なルートと失敗した URL / セレクタとともにフラグします。

## スコープに関する注意

- これは **レンダリング** を検証するものであり、リンクの正しさではありません — リンクチェックは lychee（`build-and-verify-docs` 内）が担います。両方を実行してください。
- これはアクセシビリティ監査では **ありません**。アクセシビリティの *執筆* 規約については [`markdown-accessibility.instructions.md`](../../instructions/markdown-accessibility.instructions.md) を参照してください。
- ローカル専用であり、マージをゲートしません。失敗は CI のステータスとしてではなく、PR 前に必ず修正すべきものとして扱ってください。
