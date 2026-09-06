# 執筆ガイド

これは **Copilot Workshops** の **コンテンツ執筆者およびメンテナー** 向けの入口です。ワークショップを*受講*する目的でここにたどり着いた場合は、公開サイト <https://github-samples.github.io/copilot-workshops/> にアクセスしてください。

## プロジェクト全体像のモデル

- **すべてのレッスンコンテンツはプレーンな Markdown** で、リポジトリルートの `docs/` ディレクトリ配下にあります。これが信頼できる情報源（source of truth）であり、ビルド不要で github.com 上で直接閲覧できます。
- **公開機能はオプションです。** `website/` にある Astro + Starlight サイトが、これらの Markdown ファイルから公開用の GitHub Pages サイトをビルドします（ローダーの `base: '../docs'`）。レンダリング後のページをセルフホストまたはプレビューする場合にのみ必要です。
- **再利用する散文はインラインでコピーします。** インポートベースの共有コンテンツの仕組みはありません。重複するレッスンのセクションは、後述のコンテンツ整合性のセーフティネットを使って一貫性を保ってください。

```
copilot-workshops/
├── docs/                        ← Markdown ソース。ここを編集。github.com で閲覧可能。
│   ├── README.md                ← ワークショップのランディングページ（slug: index でサイトのホームも兼ねる）
│   ├── cli/                     ← Copilot CLI レッスン（0-prerequisites.md + 番号付き演習）
│   ├── vscode/                  ← VS Code レッスン（0-prerequisites.md + 番号付き演習）
│   ├── cloud/                   ← クラウドエージェントレッスン（0-prerequisites.md + 番号付き演習）
│   ├── app/                     ← GitHub Copilot アプリレッスン（セットアップは演習 1 に統合）
│   ├── es-es/ ja-jp/ ...        ← 翻訳された各ロケールのツリー（現在は app ハーネス）
│   └── _images/                 ← スクリーンショットと図（ロケール間で共有）
├── website/                     ← オプションの Astro + Starlight 公開機能
│   ├── astro.config.mjs         ← サイト URL、ベースパス、ロケール、サイドバー
│   └── src/content.config.ts    ← コンテンツローダー（base: '../docs'）
└── .github/
    ├── copilot-instructions.md   ← AI 向け執筆ガイド（人間も読めます）
    ├── instructions/             ← ファイル種別ごとの規約（Copilot に自動適用）
    └── workflows/                ← サイトビルド + Pages デプロイの CI、およびコンテンツ整合性ワークフロー
```

## 手順ごとのレシピ

### 新しいレッスンを追加する

1. **パスと番号を決めます。** レッスンは `docs/{cli,vscode,app,cloud}/N-name.md` 配下にあります。`N` はそのパスで次に使える整数で、この番号が URL スラッグ（`/cli/3-generating-code/`）を決定します。
2. **フロントマター付きでファイルを作成します:**
   ```markdown
   ---
   title: "Exercise N - Short descriptive title"
   ---

   Body starts here.
   ```
   必須は `title` のみで、これが H1 とページタイトルになります。本文に H1 を追加しないでください — Starlight がタイトルを自動でレンダリングします。
3. **本文を書きます。** Markdown と、コールアウトには GitHub の admonition 構文（`> [!NOTE]`）を使います。後述の **スタイルの要点** を参照してください。
4. **前後のナビゲーションを追加します。** ページ下部に `[previous-lesson]` と `[next-lesson]` の参照リンクを定義し、同じパス内の隣接するレッスンを指すようにします:
   ```markdown
   [previous-lesson]: ../2-custom-instructions/
   [next-lesson]: ../4-mcp/
   ```
   そのうえで、**そのパスの他のレッスンと同じスタイル** を使って本文に表示します。パス内でスタイルを混在させないでください:
   - **散文に織り込む**（CLI パスで一般的）: ``the next step is to [create the PR][next-lesson]`` のような文でレッスンを締めくくります。
   - **明示的なナビゲーションテーブル**（cloud および VS Code パスで一般的）: フロントマターの直下と本文の末尾に、1 セルのテーブルを置きます。
     ```markdown
     | [← Previous lesson: Custom instructions][previous-lesson] |
     |:--|
     ```
     ```markdown
     | [Next lesson: Custom agents →][next-lesson] |
     |--:|
     ```
   パスの最初のレッスンでは `[previous-lesson]` を省き、最後のレッスンでは `[next-lesson]` を省きます。
5. **サイドバーに登録します。** `website/astro.config.mjs` を開き、適切な `items: []` ブロックにエントリを追加します。サイドバーは*手動で*管理されており、ファイル内の順序がそのまま受講者に表示される順序になります。
6. **プレビューして検証し、PR を作成します。** コミット前にローカルでプレビューし、検証手順を実行してください — 後述の [ビルドと検証](#building-and-verifying) を参照。CI は Astro のビルドと lychee のリンクチェックを実行し、両方が成功する必要があります。

### ランディングページ（フォルダーの `README.md`）

すべてのフォルダーのランディングページは `README.md` です。これにより、github.com でそのフォルダーを閲覧したときに直接レンダリングされます。Starlight は通常フォルダーのインデックスルートを `index.md` から導出するため、各ランディングはフロントマターに、そのルートを再現する明示的な `slug:` を持ちます:

- `docs/README.md` → `slug: index`（サイトのホーム `/`）。
- `docs/<harness>/README.md` → `slug: <harness>`（例: `slug: app` → `/app/`）。
- `docs/<locale>/README.md` → `slug: <locale>`（例: `slug: es-es` → `/es-es/`）。
- `docs/<locale>/<harness>/README.md` → `slug: <locale>/<harness>`（例: `slug: es-es/app` → `/es-es/app/`）。

新しいハーネスやロケールのランディングを追加するときは、`README.md` という名前にし、その `slug:` をフォルダーパスに一致させます。ローカライズされたランディングは、英語のスラッグではなく、必ずロケールを接頭辞に付けたスラッグを使う必要があります。

### 画像を追加する

1. **ファイルを配置します。** `docs/_images/`（画像がパス固有の場合は `cli/_images/` などパススコープのディレクトリ）に置きます。ファイル名は小文字とハイフンを使い、複数のハーネスから参照される画像には `shared-` を接頭辞として付けます。
2. **参照します。** それを利用する Markdown ページからの相対パスで参照します:
   ```markdown
   ![Description of the screenshot](../_images/my-screenshot.png)
   ```
3. **必ず代替テキスト（alt text）を含めます。** Starlight は代替テキストを必須として扱います。
4. **ローカルでプレビュー** し、画像が正しく解決されることを確認します。

### 既存のレッスンを編集する

1. **ファイルを見つけます。** `docs/` 配下で探します（公開 URL がヒントになります — `/cli/3-generating-code/` は `docs/cli/3-generating-code.md` にあります）。
2. **Markdown を編集します。** 同じ規約が適用されます — 後述の **スタイルの要点** を参照。
3. **プレビュー** は `website/` で `npm run dev` を実行します。
4. **コミット、PR、マージ** します。

### パスをまたいで散文を再利用する

同じレッスンのテキストが複数のハーネス（CLI、VS Code、Cloud）に当てはまる場合は、それを利用する各 `.md` レッスンに散文をインラインでコピーします。単一ソースのインポート層は意図的に用意していません。各レッスンは、単独の Markdown として読めて編集できるべきだからです。

コピーした散文はずれていく可能性があるため、重複するセクションを変更したときは `check-content-alignment` スキルを実行してください。差分をスキャンし、同じ更新が必要な関連コンテンツを見つけてくれます。`.github/workflows/content-alignment.md` のエージェント型ワークフローは、追加のセーフティネットとして PR 上で同じ分析を実行しますが、整合したコピーの一貫性を保つ責任は依然として執筆者にあります。

## ビルドと検証

PR を作成する前に、サイトをプレビューし、一連の検証を実行してください。正式なコマンドは [`build-and-verify-docs`](./.github/skills/build-and-verify-docs/SKILL.md) スキルにあり、以下の要約はそれを反映したものです。

Astro 開発サーバー（ホットリロード）で **プレビュー** します:

```bash
cd website
npm install
npm run dev
```

サイトは <http://localhost:4321/copilot-workshops/> で起動します。

コミット前に **検証** します:

1. **ビルド** — `cd website && rm -rf dist && npm run build`。成功する必要があります。
2. **ページ数の不変条件** — Starlight は英語と、設定された 5 つのロケールそれぞれについて 36 のワークショップルートを出力し、さらに従来のリダイレクトを追加します。これは 404 ページを除くと 217 個のビルド済み `index.html` ページに相当し、ビルドは 404 ページを含めて 218 個の HTML ファイルを報告します。
3. **リンクチェック** — ビルドされた `website/dist/` に対する lychee（オフライン）。内部リンクや画像の切れを検出します。

**CI が強制するものと、ローカルで実行するもの:** CI（`pages.yml`）は、すべての PR で **ビルド** と **lychee** のリンクチェックを実行します。Pages ビルドジョブの一部として、ブラウザ検証やコンテンツ整合性のエージェント型ワークフローは実行しません。`main` へのマージ後、`pages.yml` がサイトを GitHub Pages にデプロイします。

**整合性チェック。** ファイルやフォルダーの名前変更、スキルや instruction ファイルの追加・削除、重複する散文への変更、またはビルドの仕組みの変更を行った場合は、古くなった参照がないかも点検してください。`README.md`、`AUTHORING.md`、`.github/copilot-instructions.md` にある構造ツリーやドキュメント間のポインターは、Astro のビルドではチェックされません。[`build-and-verify-docs`](./.github/skills/build-and-verify-docs/SKILL.md) スキルに完全なチェックリストがあり、`check-content-alignment` スキルと `.github/workflows/content-alignment.md` が、整合した更新を必要とする散文の検出に役立ちます。

## スタイルの要点

簡単なチートシートです。より詳しい規約は [`.github/instructions/`](./.github/instructions/) を参照してください。

- ワークショップ内のページと外部ドキュメントには **参照スタイルのリンク** を使います。参照はページ下部で定義します:
  ```markdown
  See [Exercise 1][exercise-1] for context. The [Copilot CLI docs][cli-docs] explain.

  [exercise-1]: ../1-install-copilot-cli/
  [cli-docs]: https://docs.github.com/copilot/github-copilot-in-the-cli
  ```
- **admonition はどこでも GitHub 構文を使います** — 公開レッスン *と* リポジトリのドキュメントの両方です。`[!TYPE]` マーカーは独立した引用行に置き、本文は続く引用行に書きます:
  ```markdown
  > [!NOTE]
  > Use NOTE, TIP, IMPORTANT, WARNING, or CAUTION.
  ```
  `docs/` 配下の公開レッスンでは、remark プラグイン（`website/astro.config.mjs` で組み込み）がビルド時にこれらを Starlight のアサイドに変換します。GitHub 構文にはカスタムタイトルやネストの形式がないため、見出しは **太字の導入行**（`> **Title**` の後に空の `>` 行、その後に本文）に置き、「ネストされた」コールアウトは空行で区切った兄弟のブロッククォートとして出力します。完全な対応関係とパターンは [`markdown.instructions.md`](.github/instructions/markdown.instructions.md) を参照してください。
- リポジトリレベルの Markdown ファイル（README 群やこのファイル）では **ハードラップしません**。エディターがソフトラップします。改行は実際の構造的な区切りにのみ使います。
- **リポジトリ間のリンク**（デモアプリ）: 必ず `https://github.com/github-samples/tailspin-toys/...` を使います。*この* リポジトリ内のファイルを、あたかもテンプレートであるかのようにリンクしないでください。

## トラブルシューティング

- **「Module not found: `@astrojs/starlight/components`」** — サイトのシェルが Starlight コンポーネントをインポートしている場合は、`website/` 内で `npm install` を実行します。
- **サイドバーのエントリが表示されない** — `website/astro.config.mjs` に追加したか確認します（手動で管理されています）。
- **新しいページがビルド出力に意図せず現れる** — そのファイルがコンテンツコレクションに属しているか、また `_images/` のようなアンダースコア接頭辞の補助ディレクトリが `website/src/content.config.ts` によって引き続き除外されているかを確認します。
- **lychee がリンク切れを報告する** — 多くの場合、レッスンの名前変更によって `[ref]: ../old-name/` の定義が壊れたものです。リンク先とページ間の参照の両方を更新します。

## より詳しい規約

`.github/instructions/*.md` ファイルには、特定のファイルグロブを対象とする `applyTo` フロントマターがあります。特定の分野の詳細が必要なときに読んでください:

- [`markdown.instructions.md`](./.github/instructions/markdown.instructions.md) — Markdown 規約: ハードラップなし、admonition、見出し、ファイル名 / UI の書式、リンクスタイル。
- [`markdown-accessibility.instructions.md`](./.github/instructions/markdown-accessibility.instructions.md) — アクセシビリティ規約: 説明的なリンク、代替テキスト、見出し階層、平易な言葉づかい、入力手段に依存しない動詞（「click」ではなく **select**）。
- [`astro.instructions.md`](./.github/instructions/astro.instructions.md) — `website/` のサイトラッパー。
- [`instructions.instructions.md`](./.github/instructions/instructions.instructions.md) — instruction ファイル自体の書き方と保守方法。

再利用可能なタスクのプレイブック（ビルド / 検証、ブラウザ検証、コントリビューションフロー、コンテンツ整合性）は、`.github/skills/` 配下の **スキル** として存在します。各スキルの役割と使いどころは [スキル索引](./.github/skills/README.md) を参照してください。

AI 向けの執筆プレイブックは [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) を参照してください。
